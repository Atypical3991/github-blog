# Cassandra Architecture

Apache Cassandra is an open-source, distributed NoSQL database. It implements a partitioned wide-column storage model with eventually consistent semantics.


Apache Cassandra was designed to meet these challenges with the following design objectives in mind:
* Full multi-primary database replication
* Global availability at low latency
* Scaling out on commodity hardware
* Linear throughput increase with each additional processor
* Online load balancing and cluster growth
* Partitioned key-oriented queries
* Flexible schema



## CQL (Cassandra Query Language)

CQL allows users to organize data within a cluster of Cassandra nodes using:
* Keyspace: Specifies the replication strategy for a dataset across different datacenters. Replication refers to the number of copies stored within a cluster. Keyspaces serve as containers for tables.
* Table: Tables are composed of rows and columns. Columns define the typed schema for a single datum in a table. Tables are partitioned based on the columns provided in the partition key. Cassandra tables can flexibly add new columns to tables with zero downtime.
* Partition: Defines the mandatory part of the primary key all rows in Cassandra must have to identify the node in a cluster where the row is stored. All performant queries supply the partition key in the query.
* Row: Contains a collection of columns identified by a unique primary key made up of the partition key and optionally additional clustering keys.
* Column: A single datum with a type which belongs to a row.
* Collection types including sets, maps, and lists 
* User-defined types, tuples, functions and aggregates
* Storage-attached indexing (SAI) for secondary indexes (SAI fixes these problems by attaching the index to the SSTables (the underlying storage files) and enabling advanced query capabilities. Advantages Multi-column indexing, Scalable and Performant, Filtering with multiple indexes, Index tied to SSTables, and Range Queries supported)
* Local secondary indexes (2i)
* Single-partition lightweight transactions with atomic compare and set semantics (Cassandra handles concurrent lightweight transactions using Paxos, ensuring only one write succeeds based on the condition. All others are rejected or returned with [applied: false], preserving consistency and atomicity within a single partition.)
* (Experimental) materialized views


## Operating

Apache Cassandra configuration settings are configured in the cassandra.yaml file that can be edited by hand or with the aid of configuration management tools. Some settings can be manipulated live using an online interface, but others require a restart of the database to take effect.

Cassandra provides tools for managing a cluster. The nodetool command interacts with Cassandra’s live control interface, allowing runtime manipulation of many settings from cassandra.yaml. The auditlogviewer is used to view the audit logs. The fqltool is used to view, replay and compare full query logs.

In addition, Cassandra supports out of the box atomic snapshot functionality, which presents a point in time (PIT) snapshot of Cassandra’s data for easy integration with many backup tools. Cassandra also supports incremental backups where data can be backed up as it is written.


## Cassandra Architecture

Apache Cassandra relies on a number of techniques from Amazon’s Dynamo distributed storage key-value system. Each node in the Dynamo system has three main components:
* Request coordination over a partitioned dataset
* Ring membership and failure detection
* A local persistence (storage) engine

Cassandra primarily draws from the first two clustering components, while using a storage engine based on a Log Structured Merge Tree (LSM). In particular, Cassandra relies on Dynamo style:
* Dataset partitioning using consistent hashing
* Multi-master replication using versioned data and tunable consistency
* Distributed cluster membership and failure detection via a gossip protocol
* Incremental scale-out on commodity hardware (Cassandra allows you to add new nodes (servers) to your cluster one at a time, using inexpensive, off-the-shelf hardware, and it will automatically rebalance and scale to handle more data and traffic — without downtime or manual sharding.)
* Cassandra uses a Last-Write-Wins Element-Set conflict-free replicated data type for each CQL row, or LWW-Element-Set CRDT, to resolve conflicting mutations on replica sets.

## Consistent Hashing using a Token Ring
Cassandra maps every node to one or more tokens on a continuous hash ring, and defines ownership by hashing a key onto the ring and then "walking" the ring in one direction, similar to the Chord algorithm. The main difference of consistent hashing to naive data hashing is that when the number of nodes (buckets) to hash into changes, consistent hashing only has to move a small fraction of the keys.

For example, if we have an eight node cluster with evenly spaced tokens, and a replication factor (RF) of 3, then to find the owning nodes for a key we first hash that key to generate a token (which is just the hash of the key), and then we "walk" the ring in a clockwise fashion until we encounter three distinct nodes, at which point we have found all the replicas of that key.

You can see that in a Dynamo-like system, ranges of keys, also known as token ranges, map to the same physical set of nodes. In this example, all keys that fall in the token range excluding token 1 and including token 2 (grange(t1, t2]) are stored on nodes 2, 3 and 4.


## Multiple Tokens per Physical Node (vnodes)

Simple single-token consistent hashing works well if you have many physical nodes to spread data over, but with evenly spaced tokens and a small number of physical nodes, incremental scaling (adding just a few nodes of capacity) is difficult because there are no token selections for new nodes that can leave the ring balanced. Cassandra seeks to avoid token imbalance because uneven token ranges lead to uneven request load. For example, in the previous example there is no way to add a ninth token without causing imbalance; instead we would have to insert 8 tokens in the midpoints of the existing ranges.

The Dynamo paper advocates for the use of "virtual nodes" to solve this imbalance problem. Virtual nodes solve the problem by assigning multiple tokens in the token ring to each physical node. By allowing a single physical node to take multiple positions in the ring, we can make small clusters look larger and therefore even with a single physical node addition we can make it look like we added many more nodes, effectively taking many smaller pieces of data from more ring neighbors when we add even a single node.

Cassandra introduces some nomenclature to handle these concepts:
* Token: A single position on the Dynamo-style hash ring.
* Endpoint: A single physical IP and port on the network.
* Host ID: A unique identifier for a single "physical" node, usually present at one gEndpoint and containing one or more gTokens.
* Virtual Node (or vnode): A gToken on the hash ring owned by the same physical node, one with the same gHost ID.

Multiple tokens per physical node provide the following benefits:
* When a new node is added it accepts approximately equal amounts of data from other nodes in the ring, resulting in equal distribution of data across the cluster.
* When a node is decommissioned, it loses data roughly equally to other members of the ring, again keeping equal distribution of data across the cluster.
* If a node becomes unavailable, query load (especially token-aware query load), is evenly distributed across many other nodes.


Note that in Cassandra 2.x, the only token allocation algorithm available was picking random tokens, which meant that to keep balance the default number of tokens per node had to be quite high, at 256. This had the effect of coupling many physical endpoints together, increasing the risk of unavailability. That is why in 3.x + a new deterministic token allocator was added which intelligently picks tokens such that the ring is optimally balanced while requiring a much lower number of tokens per physical node.



## Multi-master Replication: Versioned Data and Tunable Consistency

Cassandra replicates every partition of data to many nodes across the cluster to maintain high availability and durability. When a mutation occurs, the coordinator hashes the partition key to determine the token range the data belongs to and then replicates the mutation to the replicas of that data according to the Replication Strategy.

All replication strategies have the notion of a replication factor (RF), which indicates to Cassandra how many copies of the partition should exist. For example with a RF=3 keyspace, the data will be written to three distinct replicas. Replicas are always chosen such that they are distinct physical nodes which is achieved by skipping virtual nodes if needed. Replication strategies may also choose to skip nodes present in the same failure domain such as racks or datacenters so that Cassandra clusters can tolerate failures of whole racks and even datacenters of nodes.

### Replication Strategy

Cassandra supports pluggable replication strategies, which determine which physical nodes act as replicas for a given token range. Every keyspace of data has its own replication strategy. All production deployments should use the NetworkTopologyStrategy while the SimpleStrategy replication strategy is useful only for testing clusters where you do not yet know the datacenter layout of the cluster.

* NetworkTopologyStrategy
* SimpleStrategy

### Data Versioning

Cassandra uses mutation timestamp versioning to guarantee eventual consistency of data. Specifically all mutations that enter the system do so with a timestamp provided either from a client clock or, absent a client-provided timestamp, from the coordinator node’s clock. Updates resolve according to the conflict resolution rule of last write wins. Cassandra’s correctness does depend on these clocks, so make sure a proper time synchronization process is running such as NTP.

Cassandra applies separate mutation timestamps to every column of every row within a CQL partition. Rows are guaranteed to be unique by primary key, and each column in a row resolves concurrent mutations according to last-write-wins conflict resolution. This means that updates to different primary keys within a partition can actually resolve without conflict! Furthermore the CQL collection types such as maps and sets use this same conflict-free mechanism, meaning that concurrent updates to maps and sets are guaranteed to resolve as well.

### Replica Synchronization

As replicas in Cassandra can accept mutations independently, it is possible for some replicas to have newer data than others. Cassandra has many best-effort techniques to drive convergence of replicas including Replica read repair <read-repair> in the read path and Hinted handoff <hints> in the write path.

These techniques are only best-effort, however, and to guarantee eventual consistency Cassandra implements anti-entropy repair <repair> where replicas calculate hierarchical hash trees over their datasets called Merkle trees that can then be compared across replicas to identify mismatched data. Like the original Dynamo paper Cassandra supports full repairs where replicas hash their entire dataset, create Merkle trees, send them to each other and sync any ranges that don’t match.

## Tunable Consistency
Cassandra supports a per-operation tradeoff between consistency and availability through Consistency Levels. In Cassandra, you instead choose from a menu of common consistency levels which allow the operator to pick R and W behavior without knowing the replication factor. 

Write operations are always sent to all replicas, regardless of consistency level. The consistency level simply controls how many responses the coordinator waits for before responding to the client.

For read operations, the coordinator generally only issues read commands to enough replicas to satisfy the consistency level.

## Distributed Cluster Membership and Failure Detection

The replication protocols and dataset partitioning rely on knowing which nodes are alive and dead in the cluster so that write and read operations can be optimally routed. In Cassandra liveness information is shared in a distributed fashion through a failure detection mechanism based on a gossip protocol.

### Gosip

Gossip is how Cassandra propagates basic cluster bootstrapping information such as endpoint membership and internode network protocol versions. In Cassandra’s gossip system, nodes exchange state information not only about themselves but also about other nodes they know about. This information is versioned with a vector clock of (generation, version) tuples, where the generation is a monotonic timestamp and version is a logical clock that increments roughly every second. These logical clocks allow Cassandra gossip to ignore old versions of cluster state just by inspecting the logical clocks presented with gossip messages.

Every node in the Cassandra cluster runs the gossip task independently and periodically. Every second, every node in the cluster:
* Updates the local node’s heartbeat state (the version) and constructs the node’s local view of the cluster gossip endpoint state.
* Picks a random other node in the cluster to exchange gossip endpoint state with
* Probabilistically attempts to gossip with any unreachable nodes (if one exists)
* Gossips with a seed node if that didn’t happen in step 2.

When an operator first bootstraps a Cassandra cluster, they designate certain nodes as seed nodes. Any node can be a seed node, and the only difference between seed and non-seed nodes is that seed nodes are allowed to bootstrap into the ring without seeing any other seed nodes. 

As non-seed nodes must be able to contact at least one seed node in order to bootstrap into the cluster, it is common to include multiple seed nodes, often one for each rack or datacenter. Seed nodes are often chosen using existing off-the-shelf service discovery mechanisms.

Currently, gossip also propagates token metadata and schema version (schema version in Gossip refers to how nodes track and share schema changes (like table definitions, keyspaces, indexes, etc.) across the cluster — to ensure all nodes have a consistent view of the schema.) information. This information forms the control plane for scheduling data movements and schema pulls. For example, if a node sees a mismatch in schema version in gossip state, it will schedule a schema sync task with the other nodes. As token information propagates via gossip it is also the control plane for teaching nodes which endpoints own what data.

### Ring Membership and Failure Detection

Gossip forms the basis of ring membership, but the failure detector ultimately makes decisions about if nodes are UP or DOWN. Every node in Cassandra runs a variant of the Phi Accrual Failure Detector, in which every node is constantly making an independent decision of if their peer nodes are available or not. This decision is primarily based on received heartbeat state. For example, if a node does not see an increasing heartbeat from a node for a certain amount of time, the failure detector "convicts" that node, at which point Cassandra will stop routing reads to it (writes will typically be written to hints). If/when the node starts heartbeating again, Cassandra will try to reach out and connect, and if it can open communication channels it will mark that node as available.

Cassandra will never remove a node from gossip state without explicit instruction from an operator via a decommission operation or a new node bootstrapping with a replace_address_first_boot option.

## Incremental Scale-out on Commodity Hardware

Cassandra scales-out to meet the requirements of growth in data size and request rates. Scaling-out means adding additional nodes to the ring, and every additional node brings linear improvements in compute and storage. In contrast, scaling-up implies adding more capacity to the existing database nodes. Cassandra is also capable of scale-up, and in certain environments it may be preferable depending on the deployment. Cassandra gives operators the flexibility to choose either scale-out or scale-up.

Cassandra assumes nodes can fail at any time, auto-tunes to make the best use of CPU and memory resources available and makes heavy use of advanced compression and caching techniques to get the most storage out of limited memory and storage capabilities.

## Simple Query Model
Cassandra, like Dynamo, chooses not to provide cross-partition transactions that are common in SQL Relational Database Management Systems (RDBMS).

Instead, Cassandra chooses to offer fast, consistent, latency at any scale for single partition operations, allowing retrieval of entire partitions or only subsets of partitions based on primary key filters. Furthermore, Cassandra does support single partition compare and swap functionality via the lightweight transaction CQL API.

## Simple Interface for Storing Records

 Cassandra presents a wide-column store interface, where partitions of data contain multiple rows, each of which contains a flexible set of individually typed columns. Every row is uniquely identified by the partition key and one or more clustering keys, and every row can have as many columns as needed

 This allows users to flexibly add new columns to existing datasets as new requirements surface. Schema changes involve only metadata changes and run fully concurrently with live workloads. Therefore, users can safely add columns to existing Cassandra databases while remaining confident that query performance will not degrade.
