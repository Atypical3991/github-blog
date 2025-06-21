# Cassandra

## When to Use Cassandra

1. Massive Write Throughput at Scale
    * Cassandra is optimized for high-speed writes across multiple nodes.
    * Ideal when your system ingests millions of records per second.

* Use Cases:
    * IoT sensor data
    * Logging systems
    * User activity tracking
    * Telemetry and event pipelines

2. Need for High Availability and No Single Point of Failure
    * Masterless architecture: Any node can serve any request.
    * Built-in replication and automatic failover.

* Ideal for:
    * Applications needing 24x7 uptime
    * Multi-region/multi-datacenter deployments

3. Horizontal Scalability (Scale-Out Easily)
    * Add more nodes to the cluster without downtime.
    * Cassandra scales linearly, meaning 2× the nodes → ~2× the performance.
* Great for:
    * Growing workloads
    * Systems where data volume is unpredictable or rapidly expanding
4. Decentralized Architecture (No Master Node)
    * No leader election, no single point of failure.
    * Each node has equal responsibility.
* Perfect for:
    * Multi-cloud or hybrid deployments
    * Distributed systems that can't rely on a centralized node
5. Geographically Distributed Applications
    * Cassandra supports multi-region replication.
    * Read/write locally, then syncs data across regions (eventual consistency).
* Used in:
    * Global apps with low-latency local access
    * Applications that must survive regional outages
6. Eventual Consistency Is Acceptable
    * Strong consistency is not guaranteed by default, but tunable.
    * Suitable when:
        * You're okay with a short delay in data replication
        * Read-after-write consistency is not critical
    * Examples:
        * Social media feeds
        * Notification delivery systems
        * Metrics collection
7. Time-Series or Partitioned Data Models
    * Designed for wide rows and time-series data:
        * Fast range scans on clustering keys
        * TTL (time-to-live) for automatic expiration
    * Great for:
        * Monitoring systems
        * Financial tick data
        * Weather station data
        * Messaging logs
8. You Can Design for Query-First Access Patterns
    * Cassandra requires modeling by query.
    * Denormalization and duplication are normal.
    * Schema changes are more difficult than in SQL databases.
* Use Cassandra if:
    * You know your queries in advance.
    * You can design one table per access pattern.

## When Not to Use Cassandra
| If you need...                            | Why Cassandra isn't ideal                                |
| ----------------------------------------- | -------------------------------------------------------- |
| **Ad-hoc queries**                        | Limited query flexibility; no joins or subqueries        |
| **Strong consistency (ACID)**             | Not designed for strict transactional workloads          |
| **Complex joins or aggregations**         | Not supported — requires client-side processing          |
| **Real-time analytics/BI dashboards**     | Better handled by OLAP DBs like ClickHouse or PostgreSQL |
| **Small datasets with occasional access** | Overhead of Cassandra isn't worth it                     |

## Common Cassandra Use Cases
| Industry          | Use Case                              |
| ----------------- | ------------------------------------- |
| IoT               | Sensor data, device logs              |
| Finance           | Market tick data, fraud detection     |
| Retail/E-commerce | Product views, inventory updates      |
| Social            | Feed updates, messaging metadata      |
| Telecom           | Call records, customer telemetry      |
| AdTech            | Real-time bidding, impression logs    |
| Logistics         | GPS tracking, delivery status updates |

## Datatypes
Apache Cassandra supports a wide set of data types that fall into three categories:
* Scalar types: Basic primitive types like int, text, boolean
* Collection types: Lists, sets, and maps
* User-defined types (UDTs): Custom data structures
Below is a detailed breakdown of all data types supported by Cassandra

1. Scalar Data Types (Primitives)
| Data Type          | Description                            | Example                |
|--------------------|----------------------------------------|------------------------|
| `ascii`            | ASCII string (US-ASCII)                | `'Hello'`              |
| `text` / `varchar` | UTF-8 encoded string                   | `'Hello World'`        |
| `int`              | 32-bit signed integer                  | `123`                  |
| `bigint`           | 64-bit signed integer                  | `1234567890123`        |
| `varint`           | Arbitrary-precision integer            | `12345678901234567890` |
| `float`            | 32-bit floating point                  | `12.34`                |
| `double`           | 64-bit floating point                  | `123456.789`           |
| `decimal`          | Variable-precision decimal             | `12.3456`              |
| `boolean`          | Boolean value                          | `true`                 |
| `uuid`             | Random UUID (v4)                       | `4e7f8e3c-1234-5678...`|
| `timeuuid`         | Time-based UUID (v1), sortable by time | `c3b9a040-...`         |
| `timestamp`        | Date and time with milliseconds        | `'2024-06-21 10:30'`   |
| `date`             | Date only (YYYY-MM-DD)                 | `'2024-06-21'`         |
| `time`             | Time only (HH:MM:SS.nanoseconds)       | `'10:45:30.123'`       |
| `inet`             | IP address (IPv4 or IPv6)              | `'192.168.1.1'`        |
| `blob`             | Arbitrary binary data (bytes)          | `0x1234abcd`           |
| `duration`         | Time duration (Cassandra 4.0+)         | `3mo2d10h`             |
| `smallint`         | 16-bit integer (Cassandra 4.0+)        | `1000`                 |
| `tinyint`          | 8-bit integer (Cassandra 4.0+)         | `127`                  |


2. Collection Types
| Collection  | Syntax           | Description                |
| ----------- | ---------------- | -------------------------- |
| `list<T>`   | `list<int>`      | Ordered, allows duplicates |
| `set<T>`    | `set<text>`      | Unordered, unique elements |
| `map<K, V>` | `map<text, int>` | Key-value pairs            |

3. User-Defined Types (UDTs)
Define complex, structured types — similar to a record or struct.

4. Tuple Types
Tuple = fixed-length list of typed values.

5. Frozen Collections
* Freezes the value so the entire structure is treated as a single cell.
* Required when using collections inside other collections or UDTs.

## Indexes
Apache Cassandra supports several types of indexes to allow efficient querying beyond just the primary key. However, indexing in Cassandra comes with important trade-offs, and choosing the right type is crucial for performance and scalability.


| Index Type                                  | Use Case                                         | Suitable For                            |
| ------------------------------------------- | ------------------------------------------------ | --------------------------------------- |
| **Primary Key Index**                       | Built-in, used for efficient lookups             | Main query path                         |
| **Secondary Index (2i)**                    | Querying on non-primary key columns              | Low-cardinality data within a partition |
| **SASI (SSTable Attached Secondary Index)** | Advanced indexing with filtering & range queries | More flexible queries                   |
| **Materialized Views**                      | Pre-computed alternate query paths               | Multiple access patterns                |
| **Custom Indexes**                          | Plugin-based advanced indexing (rare)            | Special use cases                       |


* Cassandra allows you to plug in custom index implementations, but:
    * Requires Java coding
    * Very rarely used in practice
    * Mostly for niche enterprise requirements


