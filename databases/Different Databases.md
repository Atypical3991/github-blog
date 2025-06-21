# Categories of Databases and When to Use Them

## Relational Databases (RDBMS)
* Examples: PostgreSQL, MySQL, Oracle, SQL Server
* Data Model: Tables (rows and columns), fixed schema
* Use When:
    * You need ACID transactions
    * You have structured data with strong relationships
    * You need JOINs, complex queries, and data integrity

* Use Cases:
    * Financial systems, ERPs, inventory, e-commerce, CRM


## NoSQL Databases
### Document Stores
* Examples: MongoDB, Couchbase, Amazon DocumentDB
* Data Model: JSON/BSON documents
* Use When:
    * You need schema flexibility
    * Nested or hierarchical data
    * Rapid development, evolving data models
* Use Cases:
    * Content management, user profiles, product catalogs

### Key-Value Stores
* Examples: Redis, DynamoDB, Riak
* Data Model: Simple key-value pairs
* Use When:
    * You need fast reads/writes
    * Caching, session storage
    * Simple data access patterns

* Use Cases:
    * Caching layers, session storage, real-time leaderboards


#### Wide Column Stores
* Examples: Apache Cassandra, ScyllaDB, HBase
* Data Model: Column families (sparse, scalable tables)
* Use When:
    * You need massive horizontal scalability
    * High write throughput
    * Denormalized, partitioned data

* Use Cases:
    * Time-series data, IoT telemetry, event logging

### Graph Databases
* Examples: Neo4j, Amazon Neptune, ArangoDB
* Data Model: Nodes and edges (relationships)
* Use When:
    * You need to model and query complex relationships
    * You need to do deep joins (e.g., friends-of-friends)
* Use Cases:
    * Social networks, fraud detection, recommendation systems

## Time-Series Databases
* Examples: InfluxDB, TimescaleDB, Prometheus
* Data Model: Time-indexed data points
* Use When:
    * You're storing metrics, logs, sensor data
    * You need efficient time-based queries, downsampling, retention policies
* Use Cases:
    * Monitoring, IoT, finance analytics, DevOps metrics

## Search Databases / Engines
* Examples: Elasticsearch, OpenSearch, Solr
* Data Model: Inverted index for text search
* Use When:
    * You need full-text search, faceted search
    * Real-time log and analytics processing
* Use Cases:
    * Log search (ELK stack), site search, auto-suggest, product search

## NewSQL Databases
* Examples: Google Spanner, CockroachDB, YugabyteDB
* Data Model: Relational (like SQL), distributed like NoSQL
* Use When:
    * You need horizontal scaling with ACID guarantees
    * Global, distributed SQL apps
* Use Cases:
    * Globally distributed apps, fintech, high-availability systems

## Multimodel Databases
* Examples: ArangoDB, OrientDB, PostgreSQL (with JSON, arrays, etc.)
* Data Model: Support for multiple paradigms — relational + document + graph
* Use When:
    * You have hybrid data needs
    * You want to reduce infrastructure sprawl


# How to Choose?
* Structured data with relationships: Relational (PostgreSQL, MySQL)
* Unstructured or evolving data: Document (MongoDB)
* High write throughput, big data: Wide Column (Cassandra, HBase)
* Caching, fast key access: Key-Value (Redis)
* Complex relationships: Graph (Neo4j)
* Time-based metrics/logs: Time-Series (InfluxDB, TimescaleDB)
* Full-text search: Search (Elasticsearch)
* Global SQL scalability: NewSQL (Spanner, CockroachDB)


