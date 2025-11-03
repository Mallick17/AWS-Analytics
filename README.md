# AWS-Analytics
## S3 vs S3 Tables
The main difference between “S3” and “S3 tables” lies in their purpose and level of optimization for analytics workloads. “S3” usually refers to the standard Amazon S3 object storage service, while “S3 tables” refers to a specialized, fully managed service built on top of S3 specifically designed for tabular data and optimized analytics performance.

<details>
    <summary>Click to view the detailed comparision of S3 and S3 Tables</summary>

### Standard S3 (Buckets)

- General-purpose object storage for any kind of unstructured data (images, backups, logs, archives, etc.).
- Objects (files) are stored in “buckets”, but S3 does not natively understand tables or schemas—it just stores files as blobs.
- Users can manually organize structured data in S3 (e.g., CSVs, Parquet), but there is no built-in support for table-level management, optimization, or analytics features.
- Performance for analytics queries can be limited, especially as data scales up and users have to manage file layout, naming, and metadata themselves.

### S3 Tables (Table Buckets)

- Purpose-built for tabular, structured datasets (like a database table: columns and rows).
- Stored in a new kind of S3 bucket type called a “table bucket”; tables are first-class resources managed by S3 itself.
- Data is stored using the Apache Iceberg format (Parquet files + metadata), enabling advanced features like schema evolution, ACID transactions, and time travel queries.
- Provides higher transactions per second (TPS) and 3x better query throughput than using self-managed Iceberg tables in standard S3 buckets.
- Integrates natively with analytics engines like Athena, Redshift, and Spark for direct SQL queries.
- Automatic table optimization: handles file compaction, metadata management, and optimizations to improve performance and lower storage costs.
- Table-level permissions, automated maintenance, and seamless integration with AWS Data Lake and Lakehouse services.

</details>

### Comparison Table

| Feature               | Standard S3 Buckets          | S3 Tables (Table Buckets)  |
|-----------------------|-----------------------------|----------------------------|
| Storage type          | General object storage     | Managed tabular storage    |
| Data structure        | Unstructured/object-based  | Columns, rows, metadata   |
| Optimization          | Manual                       | Automatic (compaction, snapshots) |
| Analytics integration | Limited/Manual               | Native for Iceberg, Athena, Redshift |
| Performance           | Depends on setup             | Up to 3x faster queries             |
| API                   | Standard S3 API              | Table-specific API, SQL support      |
| Permissions           | Bucket/object level          | Table level         |

In summary, standard S3 is best for storing any file type generically, while S3 tables are specifically optimized for structured, high-performance analytics on tabular data.

<details>
    <summary>Click to view the links of the materials</summary>

[1](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html)
[2](https://www.youtube.com/watch?v=brgh-VhN2hU)
[3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-tables.html)
[4](https://www.onehouse.ai/blog/s3-managed-tables-unmanaged-costs-the-20x-surprise-with-aws-s3-tables)
[5](https://doris.apache.org/docs/dev/lakehouse/best-practices/doris-aws-s3tables/)
[6](https://hevodata.com/learn/amazon-s3-table/)
[7](https://www.infoq.com/news/2025/01/s3-tables-bucket/)
[8](https://www.vantage.sh/blog/amazon-s3-tables)
[9](https://www.reddit.com/r/aws/comments/1h8j86w/whats_the_point_of_s3_tables/)
[10](https://dataengineeringcentral.substack.com/p/amazon-s3-tables)
[11](https://stackoverflow.com/questions/33356041/technically-what-is-the-difference-between-s3n-s3a-and-s3)

</details>
