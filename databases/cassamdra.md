# Cassandra 

Apache Cassandra stores data in tables, with each table consisting of rows and columns. CQL (Cassandra Query Language) is used to query the data stored in tables.


## Data Modeling
In Cassandra, data modeling is query-driven. The data access patterns and application queries determine the structure and organization of data which then used to design the database tables.

Data is modeled around specific queries. Queries are best designed to access a single table, which implies that all entities involved in a query must be in the same table to make data access (reads) very fast. Data is modeled to best suit a query or a set of queries. A table could have one or more entities as best suits a query. As entities do typically have relationships among them and queries could involve entities with relationships among them, a single entity may be included in multiple tables.

Since each query is backed by a table, data is duplicated across multiple tables in a process known as denormalization. Data duplication and a high write throughput are used to achieve a high read performance.


### Partitions
Apache Cassandra is a distributed database that stores data across a cluster of nodes. A partition key is used to partition data among the nodes. Cassandra partitions data over the storage nodes using a variant of consistent hashing for data distribution.

In general, the first field or component of a primary key is hashed to generate the partition key and the remaining fields or components are the clustering keys that are used to sort data within a partition. Partitioning data improves the efficiency of reads and writes. The other fields that are not primary key fields may be indexed separately to further improve query performance.

Both partition key and clustering key could be combination of more than one field in a table.

Apache Cassandra’s data model is based around designing efficient queries; queries that don’t involve multiple tables. Relational databases normalize data to avoid duplication. Apache Cassandra in contrast de-normalizes data by duplicating data in multiple tables for a query-centric data model.

### Data Model Analysis
The data model is a conceptual model that must be analyzed and optimized based on storage, capacity, redundancy and consistency. A data model may need to be modified as a result of the analysis. Considerations or limitations that are used in data model analysis include:
* Partition Size
* Data Redundancy
* Disk space

Lightweight Transactions (LWT)

The two measures of partition size are the number of values in a partition and partition size on disk. Though requirements for these measures may vary based on the application a general guideline is to keep number of values per partition to below 100,000 and disk space per partition to below 100MB.

### Meterialized View
A materialized view is a table built from data from another table, the base table, with new primary key and new properties. Changes to the base table data automatically add and update data in a MV. Different queries may be implemented using a materialized view as an MV’s primary key differs from the base table. Queries are optimized by the primary key definition.

### Patterns And Anti-Patterns
As with other types of software design, there are some well-known patterns and anti-patterns for data modeling in Cassandra.

Wide partition pattern, the essence of the pattern is to group multiple related rows in a partition in order to support fast access to multiple rows within the partition in a single query. For e.g. in available_rooms_by_hotel_date table using the hotel_id as a primary key to group room data for each hotel on a single partition, which should help searches be super fast.

Time series pattern, a series of measurements at specific time intervals are stored in a wide partition, where the measurement time is used as part of the partition key. The time series pattern is also useful for data other than measurements. Consider the example of a banking application. You’d probably be tempted to wrap a transaction around writes just to protect the balance from being updated in error. In contrast, a time series–style design would store each transaction as a timestamped row and leave the work of calculating the current balance to the application.

Any design that relies on the deletion of data is potentially a poorly performing design. The problem with this approach is that the deleted items are now tombstones <asynch-deletes> that Cassandra must scan past in order to perform read.

### Breaking Up Large Partitions
The technique for splitting a large partition is straightforward: add an additional column to the partition key. In most cases, moving one of the existing columns into the partition key will be sufficient. Another option is to introduce an additional column to the table to act as a sharding key, but this requires additional application logic.

Another technique known as bucketing is often used to break the data into moderate-size partitions. For example, you could bucketize the available_rooms_by_hotel_date table by adding a month column to the partition key, perhaps represented as an integer. The comparison with the original design is shown in the figure below. While the month column is partially duplicative of the date, it provides a nice way of grouping related data in a partition that will not get too large.


## Cassandra Query Language (CQL)
CQL is a typed language and supports a rich set of data types, including native types, collection types, user-defined types, tuple types, and custom types

### Counters
The counter type is used to define counter columns. A counter column is a column whose value is a 64-bit signed integer and on which 2 operations are supported: incrementing and decrementing (see the UPDATE statement for syntax). Note that the value of a counter cannot be set: a counter does not exist until first incremented/decremented, and that first increment/decrement is made as if the prior value was 0.
* Counters have a number of important limitations:
* They cannot be used for columns part of the PRIMARY KEY of a table.
* A table that contains a counter can only contain counters. In other words, either all the columns of a table outside the PRIMARY KEY have the counter type, or none of them have it.
* Counters do not support expiration.
* The deletion of counters is supported, but is only guaranteed to work the first time you delete a counter. In other words, you should not re-update a counter that you have deleted (if you do, proper behavior is not guaranteed).
* Counter updates are, by nature, not idemptotent. An important consequence is that if a counter update fails unexpectedly (timeout or loss of connection to the coordinator node), the client has no way to know if the update has been applied or not. In particular, replaying the update may or may not lead to an over count.

### Working with timestamps
Values of the timestamp type are encoded as 64-bit signed integers representing a number of milliseconds since the standard base time known as the epoch: January 1 1970 at 00:00:00 GMT.

Timestamps can be input in CQL either using their value as an integer, or using a string that represents an ISO 8601 date.

### Date type
Values of the date type are encoded as 32-bit unsigned integers representing a number of days with “the epoch” at the center of the range (2^31). Epoch is January 1st, 1970

For timestamps, a date can be input either as an integer or using a date string. In the later case, the format should be yyyy-mm-dd (so '2011-02-03' for instance).

### Time Type
Values of the time type are encoded as 64-bit signed integers representing the number of nanoseconds since midnight.

For timestamps, a time can be input either as an integer or using a string representing the time. In the later case, the format should be hh:mm:ss[.fffffffff] (where the sub-second precision is optional and if provided, can be less than the nanosecond)

### Duration Type
Values of the duration type are encoded as 3 signed integer of variable lengths. The first integer represents the number of months, the second the number of days and the third the number of nanoseconds. This is due to the fact that the number of days in a month can change, and a day can have 23 or 25 hours depending on the daylight saving. Internally, the number of months and days are decoded as 32 bits integers whereas the number of nanoseconds is decoded as a 64 bits integer.

### Collections
CQL supports three kinds of collections: maps, sets and lists

Collections are meant for storing/denormalizing relatively small amount of data. They work well for things like “the phone numbers of a given user”, “labels applied to an email”, etc. But when items are expected to grow unbounded (“all messages sent by a user”, “events registered by a sensor”…​), then collections are not appropriate and a specific table (with clustering columns) should be used.

Concretely, (non-frozen) collections have the following noteworthy characteristics and limitations:
* Individual collections are not indexed internally. Which means that even to access a single element of a collection, the whole collection has to be read (and reading one is not paged internally).
* While insertion operations on sets and maps never incur a read-before-write internally, some operations on lists do. Further, some lists operations are not idempotent by nature (see the section on lists below for details), making their retry in case of timeout problematic. It is thus advised to prefer sets over lists when possible.

### User-Defined Types (UDTs)
CQL support the definition of user-defined types (UDTs). Such a type can be created, modified and removed using the create_type_statement, alter_type_statement and drop_type_statement.

### Tuples
CQL also support tuples and tuple types (where the elements can be of different types). Functionally, tuples can be though as anonymous UDT with anonymous fields. 

Unlike other composed types, like collections and UDTs, a tuple is always frozen <frozen> (without the need of the frozen keyword) and it is not possible to update only some elements of a tuple (without updating the whole tuple). Also, a tuple literal should always have the same number of value than declared in the type it is a tuple of (some of those values can be null but they need to be explicitly declared as so).



## Data Definition

CQL stores data in tables, whose schema defines the layout of the data in the table. Tables are located in keyspaces. A keyspace defines options that apply to all the keyspace’s tables. The replication strategy is an important keyspace option, as is the replication factor. A good general rule is one keyspace per application. It is common for a cluster to define only one keyspace for an active application.

Both keyspace and table name should be comprised of only alphanumeric characters, cannot be empty and are limited in size to 48 characters (that limit exists mostly to avoid filenames (which may include the keyspace and table name) to go over the limits of certain file systems). By default, keyspace and table names are case-insensitive (myTable is equivalent to mytable) but case sensitivity can be forced by using double-quotes ("myTable" is different from mytable).


### Create KeySpace

The supported options are:
1. replication: The replication strategy and options to use for the keyspace (see details below).
2. durable_writes: Whether to use the commit log for updates on this keyspace (disable this option at your own risk!).

A column_definition is comprised of the name of the column and its type, restricting the values that are accepted for that column. Additionally, a column definition can have the following modifiers:
* STATIC: declares the column as a static column
* PRIMARY KEY: declares the column as the sole component of the primary key of the table


The use of static columns has the following restrictions:
* A table without clustering columns cannot have static columns. In a table without clustering columns, every partition has only one row, and so every column is inherently static
* Only non-primary key columns can be static.

#### The Primary key
Within a table, a row is uniquely identified by its PRIMARY KEY, and hence all tables must define a single PRIMARY KEY. A PRIMARY KEY is composed of one or more of the defined columns in the table.

Within a table, CQL defines the notion of a partition that defines the location of data within a Cassandra cluster. A partition is the set of rows that share the same value for their partition key.

Within a table, CQL defines the notion of a partition that defines the location of data within a Cassandra cluster. A partition is the set of rows that share the same value for their partition key.

Duplicate values in the full clustering column combination are not allowed for a given partition key.

The clustering order of a table is defined by the clustering columns. By default, the clustering order is ascending for the clustering column’s data types. For example, integers order from 1, 2, …​ n, while text orders from A to Z.

It limits how the ORDER BY clause is used in SELECT statements on that table. Results can only be ordered with either the original clustering order or the reverse clustering order.

It has a performance impact on queries. Queries in reverse clustering order are slower than the default ascending order. 

Caching optimizes the use of cache memory of a table. The cached data is weighed by size and access frequency. The caching options can configure both the key cache and the row cache for the table. The following sub-options are available:
1. keys : Whether to cache keys (key cache) for this table. Valid values are: ALL and NONE.
2. rows_per_partition: The amount of rows to cache per partition (row cache). If an integer n is specified, the first n queried rows of a partition will be cached. Valid values are: ALL, to cache all rows of a queried partition, or NONE to disable row caching.

Truncating a table permanently removes all existing data from the table, but without removing the table itself.

Cassandra WHERE caluse must contain the primary_key filter condition.
Multiple INSERT, UPDATE and DELETE can be executed in a single statement by grouping them through a BATCH statement


By default, Cassandra uses a batch log to ensure all operations in a batch eventually complete or none will (note however that operations are only isolated within a single partition).

There is a performance penalty for batch atomicity when a batch spans multiple partitions. If you do not want to incur this penalty, you can tell Cassandra to skip the batchlog with the UNLOGGED option. If the UNLOGGED option is used, a failed batch might leave the batch only partly applied.

#### DDM (Dynamic Data Masking)

Dynamic data masking (DDM) obscures sensitive information while still allowing access to the masked columns. DDM doesn’t alter the stored data. Instead, it just presents the data in its obscured form during SELECT queries. However, anyone with direct access to the SSTable files will be able to read the clear data.

A masking function can be permanently attached to any column of a table. If a masking column is defined, SELECT queries will always return the column values in their masked form. The masking will be transparent to the users running SELECT queries. The only way to know that a column is masked is to consult the table definition.

Ordinary users are created without the UNMASK permission and will see masked values. Giving a user the UNMASK permission allows them to retrieve the unmasked values of masked columns. Superusers are automatically created with the UNMASK permission, and will see the unmasked values in a SELECT query results.

User-defined functions (UDFs) can be attached to a table column. The UDFs used for masking should belong to the same keyspace as the masked table. 

## Indexing Concepts

### Primary indexing
The primary index is the partition key in Apache Cassandra. The storage engine of Apache Cassandra uses the partition key to store rows of data, and the most efficient and fast lookup of data matches the partition key.

### Storage-attached indexing (SAI)
SAI uses indexes for non-partition columns, and attaches the indexing information to the SSTables that store the rows of data. The indexes are located on the same node as the SSTable, and are updated when the SSTable is updated. SAI is the most appropriate indexing method for most use cases.

### Secondary indexing (2i)
Secondary indexing is the original built-in indexing written for Apache Cassandra. These indexes are all local indexes, stored in a hidden table on each node of a Apache Cassandra cluster, separate from the table that contains the values being indexed. The index must be read from the node. This indexing method is only recommended when used in conjunction with a partition key.


### Storage-attached indexing (SAI) concepts
Storage-Attached Indexing (SAI) is a highly-scalable, globally-distributed index for Cassandra databases.

The main advantage of SAI over existing indexes for Apache Cassandra are:
* enables vector search for AI applications
* shares common index data across multiple indexes on same table
* alleviates write-time scalability issues
* significantly reduced disk usage
* great numeric range performance
* zero copy streaming of indexes

SAI enables queries that filter based on:
vector embeddings
* AND/OR logic for numeric and text types
* IN logic (use an array of values) for numeric and text types
* numeric range
* non-variable length numeric types
* text type equality
* CONTAINs logic (for collections)
* tokenized data
* row-aware query path
* case sensitivity (optional)
* unicode normalization (optional)

SAI is deeply integrated with the storage engine of Cassandra.

SAI is also fully compatible with zero-copy streaming (ZCS). Thus, when you bootstrap or decommission nodes in a cluster, the indexes are fully streamed with the SSTables and not serialized or rebuilt on the receiving node’s end.

SAI is deeply integrated with the storage engine of the underlying database. SAI does not abstractly index tables. Instead, SAI indexes Memtables and Sorted String Tables (SSTables) as they are written, resolving the differences between those indexes at read time. 


## Materialized Views
The CREATE MATERIALIZED VIEW statement creates a new materialized view. Each such view is a set of rows which corresponds to rows which are present in the underlying, or base, table specified in the SELECT statement. A materialized view cannot be directly updated, but updates to the base table will cause corresponding updates in the view.

A view must have a primary key and that primary key must conform to the following restrictions:
* it must contain all the primary key columns of the base table. This ensures that every row of the view correspond to exactly one row of the base table.
* it can only contain a single column that is not a primary key column in the base table.

##  Functions
CQL supports 2 main categories of functions:
1. scalar functions that take a number of values and produce an output
2. aggregate functions that aggregate multiple rows resulting from a SELECT statemet

User-defined functions (UDFs) execute user-provided code in Cassandra. By default, Cassandra supports defining functions in Java.

UDFs are part of the Cassandra schema, and are automatically propagated to all nodes in the cluster. UDFs can be overloaded, so that multiple UDFs with different argument types can have the same function name.

User-defined aggregates allow the creation of custom aggregate functions. User-defined aggregates can be used in SELECT statement.

## Triggers
Cassandra supports Triggers. The below ops can be performed agaist Triggers.
1. Create Trigger
2. Drop Trigger

The actual logic that makes up the trigger can be written in any Java (JVM) language and exists outside the database. You place the trigger code in a lib/triggers subdirectory of the Cassandra installation directory, it loads during cluster startup, and exists on every node that participates in a cluster. The trigger defined on a table fires before a requested DML statement occurs, which ensures the atomicity of the transaction.

## Vector Search
Vector Search is a new feature added to Cassandra 5.0. It is a powerful technique for finding relevant content within large document collections and is particularly useful for AI applications.

## Backups

Apache Cassandra stores data in immutable SSTable files. Backups in Apache Cassandra database are backup copies of the database data that is stored as SSTable files. Backups are used for several purposes including the following:
* To store a data copy for durability
* To be able to restore a table if table data is lost due to node/partition/network failure
* To be able to transfer the SSTable files to a different machine; for portability

Types of Backups:
1. Snapshots: A hard-link copy of all SSTables at a point in time
2. Incremental Backups: Copies of SSTables that have changed since the last snapshot

## Bloom Filters
In the read path, Cassandra merges data on disk (in SSTables) with data in RAM (in memtables). To avoid checking every SSTable data file for the partition being requested, Cassandra employs a data structure known as a bloom filter.

Bloom filters are a probabilistic data structure that allows Cassandra to determine one of two possible states: - The data definitely does not exist in the given file, or - The data probably exists in the given file.

## CDC
Change data capture (CDC) provides a mechanism to flag specific tables for archival as well as rejecting writes to those tables once a configurable size-on-disk for the CDC log is reached. 

How CDC Works in Cassandra:
* When CDC is enabled on a table: Cassandra writes data mutations to special commit logs called CDC logs (in a separate location)
* These logs are in a binary format, and tools like Apache Flink, Kafka Connect, or custom scripts can process them