# AWS - S3 Table
## 1. S3 Table
Amazon S3 has long been a foundational object storage service for storing all types of unstructured and structured data. However, modern analytics, lakehouse, and large-scale data engineering workloads require structure, schema handling, transactions, and optimization — all of which **standard S3 buckets do not provide natively**.

**Amazon S3 Tables** is a new managed service that builds structured, tabular, Apache Iceberg–based tables **directly on top of S3**, providing highly optimized analytics performance without managing your own metadata, manifests, compaction, or Iceberg workflows.

---

### **2. Standard S3 vs S3 Tables (Table Buckets)**

#### **2.1 Comparison Table — S3 vs S3 Tables**

| Feature                    | Standard S3            | S3 Tables                          |
| -------------------------- | ---------------------- | ---------------------------------- |
| Storage Type               | General object storage | Managed Iceberg table storage      |
| Data Structure             | Objects (files)        | Tables (rows, columns, partitions) |
| Schema Support             | None                   | Full schema + evolution            |
| Transactions               | No                     | ACID                               |
| Snapshots / Time Travel    | No                     | Yes                                |
| Analytics Optimization     | Manual                 | Automatic                          |
| Metadata Management        | User-managed           | Fully managed                      |
| Performance                | Depends on file layout | Up to **3x faster** querying       |
| Integration with Analytics | Limited                | Native integration                 |
| Permissions                | Bucket/object level    | Table-level IAM                    |

#### **2.2 Standard S3 Buckets**

General-purpose object storage:

* Stores unstructured files: images, logs, backups, CSVs, Parquet, JSON, binaries.
* No concept of **tables**, **schemas**, **partitions**, **transactions**, **snapshots**, or **ACID** guarantees.
* For analytics, you must manually manage:

  * File format (CSV/Parquet)
  * Partition strategy
  * Folder naming
  * Performance tuning
  * Metadata cataloging
* Query engines (Athena, Spark) must scan files directly → slower at scale.

#### **2.3 S3 Tables (Table Buckets)**

- Purpose-built for tabular, structured datasets (like a database table: columns and rows).
- Stored in a new kind of S3 bucket type called a “table bucket”; tables are first-class resources managed by S3 itself.
- Data is stored using the Apache Iceberg format (Parquet files + metadata), enabling advanced features like schema evolution, ACID transactions, and time travel queries.
- Provides higher transactions per second (TPS) and 3x better query throughput than using self-managed Iceberg tables in standard S3 buckets.
- Integrates natively with analytics engines like Athena, Redshift, Amazon SageMaker Lakehouse, AWS Glue Data Catalog and Apache Spark, Flink, Hive engines for direct SQL queries.
- Automatic table optimization: handles file compaction, metadata management, and optimizations to improve performance and lower storage costs.
- Table-level permissions, automated maintenance, and seamless integration with AWS Data Lake and Lakehouse services.

> Up to **10 table buckets per region**
> Up to **10,000 tables per bucket**
> S3 automatically optimizes table data layout and metadata for analytics.

---

### **3. [Apache Iceberg — Why S3 Tables Use It](https://aws.amazon.com/what-is/apache-iceberg/)**

Apache Iceberg is an open-source table format built for data lakes. S3 Tables:

* Store data in **Parquet files**
* Store metadata using **Iceberg manifests**
* Provide high-speed reads and efficient incremental writes

<details>
    <summary>Click to view the Key Capabilities of Apache Iceberg</summary>

### Key Capabilities of Apache Iceberg**

Apache Iceberg is a modern open-source table format designed for large-scale analytics on data lakes. It provides advanced features such as ACID transactions, schema evolution, versioning, and incremental processing — all while remaining engine-agnostic and SQL-friendly.

#### **1. SQL Familiarity**

Iceberg fully supports SQL-based table operations.
Anyone familiar with SQL can create, modify, query, and manage Iceberg tables without learning new languages or frameworks. This makes Iceberg easy to adopt for analysts, engineers, and developers.

#### **2. Strong Data Consistency**

Iceberg ensures that all readers and writers see a consistent view of the dataset.
It uses ACID transactions so that concurrent operations do not conflict, guaranteeing reliable data reads and writes across distributed systems.

#### **3. Flexible Data Structure (Schema Evolution)**

Iceberg allows seamless and safe schema changes, including:

* Adding columns
* Renaming columns
* Removing columns

These operations do not require rewriting existing data and do not break queries or pipelines.

#### **4. Data Versioning and Time Travel**

Iceberg maintains snapshots of table states over time.
This enables:

* Querying historical versions (time travel)
* Comparing old and new data
* Auditing changes after updates or deletes

Snapshots make rollback and historical analysis simple and efficient.

#### **5. Cross-Platform Compatibility**

Iceberg works across multiple engines and storage systems.
It integrates with:

* Apache Spark
* Apache Flink
* Apache Hive
* Presto/Trino
* AWS Athena, Redshift, EMR
  This flexibility allows Iceberg tables to be used in any modern data lake or lakehouse environment.

#### **6. Incremental Processing (CDC Support)**

Iceberg supports efficient incremental data processing.
Instead of scanning entire datasets, engines can read only:

* New data
* Modified data
* Deleted data

This reduces compute cost and improves job performance for CDC, ETL, and streaming workloads.

---

#### **7. Maintenance Configuration Limitation**

Certain Iceberg maintenance settings are **incompatible**:

* `history.expire.max-snapshot-age-ms`
* `history.expire.min-snapshots-to-keep`

These two properties cannot be used together because they represent conflicting snapshot retention rules.
One controls retention by **age**, the other by **count** — so only one method should be used per table.

</details>

---

### **4. Analytics Integrations of S3 Tables**

S3 Tables are automatically discoverable by analytics services via Glue Data Catalog.

##### **Native Integrations**

* **Athena SQL**
* **Amazon Redshift Lakehouse**
* **Amazon SageMaker Lakehouse**
* **Amazon EMR**
* **AWS Glue ETL**
* **Apache Spark / Flink / Hive**
* **QuickSight**

These engines can directly query, snapshot, merge, compact, or time-travel the Iceberg tables.

---

### **5. S3 Tables Pricing (Mumbai Region)**

#### **5.1 Storage Charges**

| Tier              | Price (per GB per month)     |
| ----------------- | ---------------------------- |
| First 50 TB       | **$0.0288**                  |
| Next 450 TB       | **$0.0276**                  |
| Over 500 TB       | **$0.0265**                  |
| Object Monitoring | **$0.025 per 1,000 objects** |

#### **5.2 Requests Pricing**

| Request Type      | Price                          |
| ----------------- | ------------------------------ |
| PUT / POST / LIST | **$0.005 per 1,000 requests**  |
| GET / Others      | **$0.0004 per 1,000 requests** |

**Example: 1,003 PUTs/day × 30 days → 30,090/month**
Cost = **30,090 × $0.005 / 1000 = $0.15**

#### **5.3 Compaction Pricing**

| Compaction Type                    | Price                        |
| ---------------------------------- | ---------------------------- |
| Objects processed                  | **$0.002 per 1,000 objects** |
| Data processed (binpack – default) | **$0.005 per GB**            |
| Data processed (Sort / Z-order)    | **$0.01 per GB**             |

#### **5.4 Data Transfer Out (Slabs)**

| Slab     | Price          |
| -------- | -------------- |
| 10.24 TB | $0.1093 per GB |
| 40.96 TB | $0.085 per GB  |
| 102.4 TB | $0.082 per GB  |
| 870.4 TB | $0.08 per GB   |

#### **5.5 Full Cost Calculation Model**

> Total = S3 Tables storage charge + PUT request charge + GET request charge + Object monitoring charge + Compaction (objects + data processed) + Data outbound charges

---

### **6. Do Applications Writing Directly to S3 Tables Trigger Compaction Costs?**

#### **Short Answer:** **Yes — eventually.**

But **not immediately with every PUT**.

#### 1. When your application writes data using:

* S3 PUT APIs
* Iceberg compatible writers
* Spark / Flink / Athena INSERT statements

It writes **files** into table buckets.

#### 2. These incoming files can be:

* Small
* Unoptimized
* Many-in-number

Over time, this leads to:

* High metadata overhead
* Slower queries
* More manifest files

#### 3. To fix this, **S3 Tables automatically runs maintenance jobs**, including:

File compaction
Metadata cleanup
Snapshot expiration
Small-file merging

These **maintenance operations** are what trigger:

**Compaction object charges**
**Compaction GB processed charges**

#### **Therefore:**

> **Writing data directly to S3 tables WILL eventually incur compaction charges, because automatic optimization is part of the service.**

#### **When is compaction triggered?**

* Lots of small files are created
* Too many data files per partition
* Too many metadata files
* Scheduled maintenance windows
* Query engines require optimization

#### **You cannot avoid compaction charges entirely**, but you can reduce them by:

* Writing larger Parquet files (64–512 MB)
* Reducing small/fragmented writes
* Using batching or micro-batching
* Using Spark/Flink optimized writers

<details>
    <summary>Click to view the detailed Explaination</summary>

When you store data in Amazon S3 Tables using put requests (uploading data files), the S3 Tables service automatically performs compaction in the background to optimize storage and query efficiency. Here is how compaction occurs:

### How Compaction Occurs in S3 Tables

1. **Granular Writes Create Many Small Files:**  
   Each put request often creates a small file (or object) in the table, especially in transactional or streaming workloads where data arrives continuously and in small chunks.

2. **Small Files Impact Query Performance:**  
   Large numbers of small files increase the overhead for query engines, needing multiple reads and scans, which degrade performance.

3. **Automatic Background Compaction:**  
   S3 Tables automatically combines many smaller files into fewer, larger files during compaction. This process is transparent to users and requires no manual intervention.

4. **Target File Size and Strategies:**  
   - By default, S3 Tables aim to compact files to about 512 MB in size, but this target can be tuned between 64 MB and 512 MB via AWS CLI configuration.  
   - Different compaction strategies are supported including binpack (default), sort, and z-order compaction for optimized query patterns on large-scale datasets.

5. **Compacted Files Form Latest Table Snapshot:**  
   Files created by compaction become the latest snapshot of the table, ensuring data remains current and efficiently organized for queries.

6. **Benefits:**  
   - Improved query speed due to fewer file scans and higher data read throughput.  
   - Reduced storage overhead by minimizing metadata and file fragmentation.  
   - Reduced operational complexity as manual compaction management is avoided.

### Summary of the Compaction Process

| Step                        | Description                                        |
|-----------------------------|--------------------------------------------------|
| Data Upload                 | Put requests add small files to the table        |
| Performance Impact          | Many small files degrade query performance       |
| Automatic Compaction        | Background process merges small files into bigger ones |
| Configurable Target Size    | Default 512 MB per file, adjustable via CLI      |
| Compaction Strategies       | Binpack (default), sort compaction, z-order compaction |
| Final Outcome              | Latest snapshot with optimized file structure    |

This automatic compaction in S3 Tables helps maintain efficient and performant data access for large-scale analytics workloads without user intervention or additional infrastructure.

[How amazon s3 tables uses compaction](https://aws.amazon.com/blogs/storage/how-amazon-s3-tables-use-compaction-to-improve-query-performance-by-up-to-3-times/)
[amazon-s3-tables-reduce-compaction-costs](https://aws.amazon.com/about-aws/whats-new/2025/07/amazon-s3-tables-reduce-compaction-costs/)
[S3 Tables](https://www.onehouse.ai/blog/s3-managed-tables-unmanaged-costs-the-20x-surprise-with-aws-s3-tables)
[Amazon s3 iceberg compaction](https://www.infoq.com/news/2025/07/amazon-s3-iceberg-compaction/)
[Amazon s3 Table](https://hevodata.com/learn/amazon-s3-table/)
[why-amazon-s3-tables-is-a-game-changer-for-transactional-data-lakes](https://www.granica.ai/blog/why-amazon-s3-tables-is-a-game-changer-for-transactional-data-lakes)
[data-analytics/spark-operator-s3tables](https://awslabs.github.io/data-on-eks/docs/blueprints/data-analytics/spark-operator-s3tables)
[small-file-problem-s3](https://www.upsolver.com/blog/small-file-problem-s3)
 
</details>

---

### **7. RDS vs S3 Tables**

| Feature        | RDS                          | S3 Tables                          |
| -------------- | ---------------------------- | ---------------------------------- |
| Type           | Relational Database          | Lakehouse Table Storage            |
| Optimized For  | OLTP                         | OLAP                               |
| Schema         | Strict                       | Flexible & Evolvable               |
| Transactions   | Strong ACID                  | ACID (Iceberg)                     |
| Query Type     | Row-based SQL                | Columnar analytics SQL             |
| Concurrency    | High                         | High for analytics, not OLTP       |
| Scaling        | Vertical & Read replicas     | Horizontal to petabytes            |
| Storage Format | Database engine format       | Parquet + Iceberg                  |
| Cost Model     | Compute + Storage            | Storage + Requests + Compaction    |
| Time Travel    | Point-in-time backups        | Built-in snapshots                 |
| When to Use    | Real-time apps, transactions | Analytics, BI, ML, ETL, data lakes |
| Integration    | App-level                    | Analytics engines                  |

<details>
    <summary>Click to view the key distinctions</summary>

##### **Key Distinctions**

##### **RDS** is for:

* High-speed transactional workloads
* Low-latency reads/writes
* Real-time applications
* Banking, inventory, order processing

##### **S3 Tables** is for:

* Analytics
* BI and dashboards
* Large-scale data storage
* ML training
* Time-travel analysis
* ETL pipelines
* Lakehouse architectures

> **RDS stores rows; S3 Tables store columnar data.**
> **RDS cannot scale to PBs affordably; S3 Tables are designed for PB–EB scale.**

</details>

---

### **8. S3 Tables — Behavior Summary**

##### **1. Automatic Maintenance**

* Merges small files
* Rewrites partitions
* Cleans metadata
* Handles snapshot pruning

##### **2. Optimized Query Performance**

* Pushdown filters
* Column pruning
* Partition pruning
* Iceberg metadata skipping

##### **3. High Transaction Throughput**

* Designed for large-scale analytics ingestion
* Supports parallel writes

##### **4. Full Iceberg Semantics**

* ACID
* Schema evolution
* Time travel
* Incremental scans

##### **5. Strong Integrations**

* Athena: direct SQL
* Redshift: lakehouse analytics
* Spark/Flink: streaming and batch
* Glue: ETL automation

<details>
    <summary>Click to view the links of Online References (All Articles Included)</summary>

1. [https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html)
2. [https://www.youtube.com/watch?v=brgh-VhN2hU](https://www.youtube.com/watch?v=brgh-VhN2hU)
3. [https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-tables.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-tables.html)
4. [https://www.onehouse.ai/blog/s3-managed-tables-unmanaged-costs-the-20x-surprise-with-aws-s3-tables](https://www.onehouse.ai/blog/s3-managed-tables-unmanaged-costs-the-20x-surprise-with-aws-s3-tables)
5. [https://doris.apache.org/docs/dev/lakehouse/best-practices/doris-aws-s3tables/](https://doris.apache.org/docs/dev/lakehouse/best-practices/doris-aws-s3tables/)
6. [https://hevodata.com/learn/amazon-s3-table/](https://hevodata.com/learn/amazon-s3-table/)
7. [https://www.infoq.com/news/2025/01/s3-tables-bucket/](https://www.infoq.com/news/2025/01/s3-tables-bucket/)
8. [https://www.vantage.sh/blog/amazon-s3-tables](https://www.vantage.sh/blog/amazon-s3-tables)
9. [https://www.reddit.com/r/aws/comments/1h8j86w/whats_the_point_of_s3_tables/](https://www.reddit.com/r/aws/comments/1h8j86w/whats_the_point_of_s3_tables/)
10. [https://dataengineeringcentral.substack.com/p/amazon-s3-tables](https://dataengineeringcentral.substack.com/p/amazon-s3-tables)
11. [https://stackoverflow.com/questions/33356041/technically-what-is-the-difference-between-s3n-s3a-and-s3](https://stackoverflow.com/questions/33356041/technically-what-is-the-difference-between-s3n-s3a-and-s3)

</details>

---

# Debezium
### Debezium MySQL Connector Configuration Overview
The Debezium MySQL connector captures row-level changes from MySQL databases (including Amazon RDS for MySQL) by reading the binary log (binlog). Configurations are set as key-value pairs when registering the connector via Kafka Connect REST API or properties files. All properties are optional unless marked required, with sensible defaults for most. 

<details>
    <summary>Click to view the Key Points and Parameters</summary>

**Key Points**:
- **Types**: Primarily strings (e.g., for lists/hosts), integers (e.g., timeouts), booleans, or enums (option lists).
- **Validation**: Kafka Connect performs type and value validation at startup. Invalid values (e.g., wrong type, out-of-range, malformed regex) typically cause a `ConfigException` or `ValidationException`, preventing the connector from starting. Some (e.g., regex mismatches) may log warnings and partially succeed but lead to skipped tables/events or data inconsistencies.
- **Defaults and Limits**: Defaults ensure basic functionality; limits are often positive integers, valid regex, or enums. Exceeding limits (e.g., oversized timeouts) may cause timeouts or resource exhaustion.
- **Custom/Deprecated**: Some properties support custom extensions (e.g., via class names); deprecated ones log warnings and may be removed in future versions.
- **Impact of Invalid/Other Params**: Undefined params use defaults. Overriding with invalid values halts startup or causes runtime errors (e.g., connection failures). Extra undefined params are ignored (no effect).

For a full, up-to-date list, refer to the [official Debezium docs](https://debezium.io/documentation/reference/connectors/mysql.html).

### RDS-Specific Considerations
Debezium works seamlessly with Amazon RDS MySQL (and Aurora MySQL), as it's MySQL-compatible. Key differences:
- **Locking**: RDS/Aurora doesn't support global read locks (`FLUSH TABLES WITH READ LOCK`), so the connector uses **table-level locks** (`LOCK TABLES`) during snapshots. The Debezium user must have `LOCK TABLES` privilege.
- **Binlog**: Ensure binlog is enabled in RDS parameter group (`binlog_format=ROW`, `log_bin=1`). RDS has limits on binlog retention (default 1 day; configurable up to 35 days via `binlog_expire_logs_seconds`).
- **GTIDs**: Supported for multi-master/replicas; enable via RDS params (`gtid_mode=ON`).
- **No Additional Params**: Use standard MySQL properties; no RDS-exclusive configs. Test snapshots thoroughly, as table locks can briefly block writes on large tables.
- **Limits**: RDS I/O throughput (e.g., Provisioned IOPS) affects snapshot speed; high-traffic DBs may need `snapshot.mode=never` after initial sync to avoid locks.
- **Invalid Config Impact**: Same as MySQL; e.g., missing `LOCK TABLES` privilege causes snapshot failures with "Access denied" errors.

If using RDS, monitor CloudWatch for binlog disk usage and connector lag.

### Required Configuration Properties
These must be set for basic connectivity and operation.

| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `connector.class` | string | N/A | Java class for the connector. | Must be `io.debezium.connector.mysql.MySqlConnector`. | Startup failure with `ClassNotFoundException` or validation error. |
| `database.hostname` | string | N/A | MySQL/RDS server IP/hostname. | Valid IP or resolvable hostname. | Connection timeout/failure; connector won't start. |
| `database.port` | int | 3306 | MySQL/RDS server port. | 1-65535. | Connection failure; e.g., invalid port logs "Connection refused." |
| `database.user` | string | N/A | Username for connector (with REPLICATION SLAVE, etc., privileges). | Valid MySQL user. | Authentication error (401); connector fails to connect. |
| `database.password` | string (password) | N/A | Password for the user. | Secure string (externalize via secrets). | Authentication error; same as above. |
| `database.server.id` | int | N/A | Unique numeric ID for this connector instance (avoids binlog conflicts). | Positive integer (e.g., 184054); unique across cluster. | Replication slot conflict; binlog read fails with "Server ID collision." |
| `topic.prefix` | string | N/A | Prefix for all topics (e.g., `dbserver1`). | Alphanumeric + `_` (starts with letter); used for events like `<prefix>.<db>.<table>`. | Topic creation fails if invalid Kafka name; replaces invalid chars with `_` (may cause duplicates). |

### Common/Optional Configuration Properties
These control snapshots, filtering, data handling, etc. Grouped by category.

#### Connection and Heartbeat
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `connect.timeout.ms` | int | 30000 | Max ms to wait for DB connection. | Positive integer (ms). | Connection hangs indefinitely if <=0; validation error. |
| `heartbeat.interval.ms` | int | 0 (disabled) | Ms between heartbeat events to topic to detect failures. | Positive integer (ms); 0 disables. | If <=0, no heartbeats; may miss offsets on outage. |
| `heartbeat.topics.prefix` | string | `<topic.prefix>.heartbeat` | Prefix for heartbeat topics. | Valid topic prefix. | Heartbeat events not emitted; monitoring fails. |

#### Snapshot Configuration
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `snapshot.mode` | string (enum) | `initial` | Controls initial/current snapshot behavior (e.g., data/schema inclusion). | Enums: `initial` (full snapshot + stream), `schema_only` (deprecated; schema only), `initial_only` (snapshot then stop), `never` (no snapshot, stream from now), `when_needed` (snapshot if offsets missing), `recovery` (rebuild schema history), `always` (snapshot every run), `custom` (custom impl). | Validation error; e.g., unknown enum prevents startup. May trigger unwanted snapshots or miss data. |
| `snapshot.delay.ms` | long | 0 | Ms to wait before snapshot (for load balancing). | Non-negative integer (ms). | If <0, validation error; snapshot starts immediately. |
| `snapshot.fetch.size` | int | 2000 | Max rows fetched per snapshot query. | Positive integer. | If <=0, excessive memory use or validation error. |
| `snapshot.lock.timeout.ms` | long | 10000 | Ms to wait for snapshot locks. | Positive integer (ms). | Lock timeout errors if <=0; snapshot fails. |
| `snapshot.select.statement.overrides` | string | N/A | Custom SELECT for specific tables in snapshots (e.g., `db.table:SELECT * FROM db.table WHERE id > 1000`). | Comma-separated `db.table:SELECT stmt`. | SQL syntax error skips table; incomplete snapshot. |
| `snapshot.new.tables` | string (enum) | N/A | Behavior for new tables post-snapshot. | Enums: `include` (snapshot new tables), `exclude` (ignore). | Unknown enum: validation error; may miss new tables. |

#### Table and Column Filtering
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `database.include.list` | string | N/A | Comma-separated regex for DBs to include. | Anchored regex (e.g., `inventory`). | Malformed regex: validation error; captures all/wrong DBs. |
| `database.exclude.list` | string | N/A | Comma-separated regex for DBs to exclude. | Anchored regex; mutually exclusive with include. | Conflict with include: validation error. |
| `table.include.list` | string | N/A | Comma-separated regex for tables (e.g., `db.table`). | Anchored regex; case-sensitive. | Malformed: skips tables, logs error. |
| `table.exclude.list` | string | N/A | Comma-separated regex for tables to exclude. | Anchored regex. | Malformed: captures unintended tables. |
| `column.include.list` | string | N/A | Comma-separated regex for columns (e.g., `db.table.col`). | Anchored regex; mutually exclusive with exclude. | Conflict: validation error. |
| `column.exclude.list` | string | N/A | Comma-separated regex for columns to exclude. | Anchored regex. | Malformed: includes sensitive columns. |

#### Data Type Handling
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `bigint.unsigned.handling.mode` | string (enum) | `precise` | How to handle BIGINT UNSIGNED. | `precise` (BigDecimal), `long` (long, may overflow). | Unknown: validation error; data loss on overflow. |
| `binary.handling.mode` | string (enum) | `bytes` | Binary column representation. | `bytes`, `base64`, `base64-url-safe`, `hex`. | Unknown: validation error; garbled binary data. |
| `decimal.handling.mode` | string (enum) | `precise` | DECIMAL/NUMERIC handling. | `precise` (BigDecimal), `string`, `double`. | Unknown: precision loss. |
| `time.precision.mode` | string (enum) | `adaptive` | Temporal precision (e.g., microseconds). | `adaptive`, `connect`. | Unknown: reduced precision in events. |
| `event.deserialization.failure.handling.mode` | string (enum) | `fail` | Handle binlog deserialization errors. | `fail` (stop), `warn` (log/skip), `ignore` (silent skip). | Unknown: defaults to `fail`; data loss if ignore misused. |

#### Schema History and Signaling
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `schema.history.internal.kafka.bootstrap.servers` | string | N/A | Kafka brokers for schema history topic. | Comma-separated hosts:ports. | History storage failure; schema drift on restart. |
| `schema.history.internal.kafka.topic` | string | N/A | Topic for DDL history (1 partition only). | Valid topic name; single partition required. | Multi-partition: history corruption; connector errors. |
| `signal.data.collection` | string | N/A | Table for ad-hoc snapshot signals (e.g., `db.signal_table`). | `db.table` format. | Signals ignored; no ad-hoc snapshots. |

#### Performance and Advanced
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `heartbeat.interval.ms` | int | 0 | Heartbeat frequency. | >=0 ms. | No heartbeats if <0; offset loss risk. |
| `max.batch.size` | int | 2048 | Max events per batch. | Positive integer. | If <=0, unbounded batches; memory exhaustion. |
| `max.queue.size` | int | 8192 | Buffer queue size. | Positive integer. | If <=0, unlimited queue; OOM on high load. |
| `poll.interval.ms` | int | 500 | Binlog polling interval. | Positive integer (ms). | High CPU if too low; lag if too high. |
| `snapshot.fetch.size` | int | 2000 | Snapshot batch fetch size. | Positive integer. | Memory issues if too high. |

#### Converters and Transformations (Examples)
| Property Name | Type | Default | Description | Limits/Valid Values | Invalid Value Behavior |
|---------------|------|---------|-------------|---------------------|------------------------|
| `key.converter` | string | N/A | Kafka key serializer (e.g., JsonConverter). | Valid class (e.g., `org.apache.kafka.connect.json.JsonConverter`). | Serialization failure; events not produced. |
| `value.converter` | string | N/A | Kafka value serializer. | Valid class. | Same as above. |
| `transforms` | string | N/A | SMT chain (e.g., `unwrap`). | Comma-separated SMT names. | Transformation fails; malformed events. |

### Additional Notes
- **Total Properties**: ~50+ in full docs; above covers core ones. For exhaustive list, see Debezium reference.
- **Other Params**: Ignored if undefined. Custom params (e.g., prefixed like `snapshot.custom.*`) require matching extensions; otherwise, ignored or validation error.
- **Limits Overview**: Timeouts (ms, positive ints); sizes (positive ints, e.g., 1024 rows); regex (anchored, Java flavor); enums (case-sensitive).
- **Error Handling**: Most invalid configs cause immediate startup failure. Runtime issues (e.g., bad regex) log warnings and degrade gracefully (e.g., skip tables).
- **Best Practices for RDS**: Use IAM DB auth if possible; monitor binlog retention to avoid offset loss.

</details>

## Debezium Setup
#### Step 1: Create Custom Config for Binlog + Local Infile
Your shown `my.cnf` is the default (no binlog; `log_bin` commented). We'll create a snippet that gets included via `/etc/mysql/conf.d/` (as per `!includedir` in the default).

```
mkdir -p ~/mysql-config
cat > ~/mysql-config/binlog.cnf << EOF
[mysqld]
# Debezium requirements
log-bin=mysql-bin
binlog_format=ROW
server-id=184054  # Unique; matches tutorial

# From your steps (enable LOAD DATA if needed)
local-infile=1

# Optional: Retain binlogs longer
expire_logs_days=10
EOF
```
- This overrides/adds only what's needed; default remains intact.

#### Step 2: Start the New Container with Mounts
```
docker run --name mysql-container \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -d -p 3306:3306 \
  -v ~/mysql-config/binlog.cnf:/etc/mysql/conf.d/binlog.cnf \
  mysql:8.0
```
- `-v .../binlog.cnf:/etc/mysql/conf.d/binlog.cnf`: Enables binlog on startup (included automatically).
- `-v .../mysql-data-extract:/var/lib/mysql`: Bind-mounts your extracted data dir (persistent; changes survive restarts).
- Wait for startup (~30-60s; MySQL recovers indexes):
  ```
  docker logs -f mysql-container
  ```
  - Look for: "ready for connections", no errors about datadir (it'll use the mounted one).
  - Ctrl+C to stop following.

#### Step 3: Follow the steps to [Import the DB to the MySQL](https://github.com/Mallick17/SQL-NoSQL#step-by-step-guide-to-import-the-employees-database-into-a-mysql-container)

#### Step 4: Verify Everything
1. **Container Running**:
   ```
   docker ps  # mysql-container should be UP
   ```

2. **Data Intact** (connect and query):
   ```
   mysql -u root -p
   USE employees;
   SHOW TABLES;
   ```
   - Lists `employees`, `salaries`, etc.
   ```
   SELECT COUNT(*) FROM employees;
   ```
   - ~300,024 rows.

3. **Binlog Enabled**:
   ```
   SHOW MASTER STATUS;
   ```
   - Shows `File: mysql-bin.000001` (or similar), `Position` > 0.
   ```
   SHOW GLOBAL VARIABLES LIKE 'binlog_format';
   ```
   - `Value: ROW`.
   ```
   SHOW GLOBAL VARIABLES LIKE 'local_infile';
   ```
   - `Value: ON`.

4. **Config Applied** (optional check):
   ```
   cat /etc/mysql/conf.d/binlog.cnf
   ```
   - Shows your snippet.

#### Step 5: Create Debezium User (If Not Already)
```
docker exec -it mysql-container mysql -u root -p=rootpass
```
In MySQL:
```
CREATE USER IF NOT EXISTS 'debezium'@'%' IDENTIFIED BY 'dbz';
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%';
GRANT ALL PRIVILEGES ON employees.* TO 'debezium'@'%';
FLUSH PRIVILEGES;
EXIT;
```
- Test connection: `mysql -u debezium -p -e "USE employees; SELECT COUNT(*) FROM employees LIMIT 1;"`
