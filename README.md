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

### **3. Apache Iceberg — Why S3 Tables Use It**

Apache Iceberg is an open-source table format built for data lakes. S3 Tables:

* Store data in **Parquet files**
* Store metadata using **Iceberg manifests**
* Provide high-speed reads and efficient incremental writes

##### **Iceberg Benefits**

SQL-friendly
ACID transactions
Schema evolution (add/remove/rename columns)
Time-travel & versioning
Incremental processing (CDC)
Compatible with Spark, Flink, Presto, Hive, Redshift, Athena

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

Total =
S3 Tables storage charge
PUT request charge
GET request charge
Object monitoring charge
Compaction (objects + data processed)
Data outbound charges

---

### **6. Do Applications Writing Directly to S3 Tables Trigger Compaction Costs?**

#### **Short Answer:** **Yes — eventually.**

But **not immediately with every PUT**.

<details>
    <summary>Click to view detailed explaination</summary>

### **Detailed Explanation**

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
