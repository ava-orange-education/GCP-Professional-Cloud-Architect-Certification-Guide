# Query Optimization in Action
Let’s say your query looks as follows:
```SQL
SELECT COUNT(*) 
FROM `web_logs.analytics`
WHERE event_date = '2025-03-15'
  AND country = 'US'
  AND device_type = 'mobile';
```
With partitioning by event_date and clustering by country and device_type, BigQuery only scans the March 15 partition and quickly zooms into rows for US mobile users—saving time and cost.
You can define partitioning and clustering while creating the table:

```SQL
CREATE TABLE `web_logs.analytics` (
  event_date DATE,
  user_id STRING,
  page_url STRING,
  country STRING,
  device_type STRING
)
PARTITION BY event_date
CLUSTER BY country, device_type;
```
You can also create partitioned tables using ingestion time like thisas follows:

```SQL
CREATE TABLE `web_logs.analytics`
(
  user_id STRING,
  page_url STRING,
  country STRING,
  device_type STRING
)
PARTITION BY _PARTITIONTIME
CLUSTER BY country, device_type;
```

Thus, to summarize:

- Partitioning reduces the amount of data scanned by focusing on relevant date ranges.
- Clustering improves query performance on frequently filtered or grouped columns.
- Use both together for maximum efficiency.

Together, they make BigQuery smarter and more cost-effective—especially for high-volume event data likesuch as logs, IoT signals, or transaction histories.

# Managing Data Lifecycle in BigQuery: Table and Partition Expiration
As datasets grow, it’sit is important to avoid storing stale or obsolete data. BigQuery helps manage storage costs and data hygiene through table and partition expiration settings. These settings let you automatically delete data after a certain period—no manual cleanup is required.
This is especially useful for:
- Log or event data that becomes irrelevant after a few weeks or months
- Temporary tables used in data pipelines or experiments
- Managing retention policies for compliance and cost control

## Table Expiration
When you create a BigQuery table, you can specify a default expiration time. Once the expiration time passes, BigQuery automatically deletes the table. This helps avoid clutter and ensures short-lived datasets don’t not hang around longer than necessary.
Key Details:
- Expiration is defined in seconds from the time of creation.
- You can set expiration at table creation or modify it later.
- If no expiration is set, the table will persist until explicitly deleted.

The following code sample shows, how you can set an expiration to a table to seven days from today:
```SQL
CREATE TABLE `project_id.dataset_id.temp_events`
(
  user_id STRING,
  event_type STRING,
  event_time TIMESTAMP
)
OPTIONS (
  expiration_timestamp = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
);
```
This creates a table that will automatically expire and be deleted 7 days after creation.


## Partition Expiration
If your table is partitioned, you can set a default partition expiration to automatically delete individual partitions after a specific time window—relative to their ingestion or partition key value.
This is ideal for use cases likesuch as:
- Keeping only the last 30 days of logs
- Auto-expiring old data without touching recent entries
For example:
```SQL
CREATE TABLE `project_id.dataset_id.partitioned_logs`
(
  event_date DATE,
  user_id STRING,
  action STRING
)
PARTITION BY event_date
OPTIONS (
  partition_expiration_days = 30
);
```

In the preceding query, each partition will expire 30 days after its event_date. So, for example, the partition for 2025-02-15 will be automatically deleted on 2025-03-17.

Best Practices:
- Use expiration settings for transient or time-bounded data likesuch as logs, testing datasets, or temporary processing stages.
- Monitor your expiration policies so that critical data isn’t not deleted unintentionally.
- Use alerts or logging (via Cloud Monitoring) if you need visibility when tables or partitions are auto-deleted.

# Loading Data Example (CSV from Cloud Storage)

The following SQL command loads CSV data from a Cloud Storage URI into a partitioned BigQuery table. Other formats just require switching the format = value.

```SQL
LOAD DATA INTO `project.dataset.sales_data`
FROM FILES (
  format = 'CSV',
  uris = ['gs://my-bucket/data/sales.csv']
)
WITH PARTITION COLUMNS(event_date);
```

For efficient loading:
- Prefer Avro, Parquet, or ORC for large-scale production pipelines.
- Use CSV and JSON for prototyping or quick loads.
- Always validate schemas before loading.
- Compress your files (e.g., GZIP) to reduce storage and improve load speed.
