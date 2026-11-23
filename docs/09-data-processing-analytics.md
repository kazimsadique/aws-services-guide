# 09 — Data Processing & Analytics

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| — | [The modern AWS data-lake flow](#the-modern-aws-data-lake-flow) | Overview |
| 9.1 | [Amazon S3 as a data lake](#91-amazon-s3-as-a-data-lake) | 🟡 Intermediate |
| 9.2 | [Amazon Kinesis](#92-amazon-kinesis) | 🔴 Advanced |
| 9.3 | [AWS Glue](#93-aws-glue) | 🟡 Intermediate |
| 9.4 | [Amazon EMR (Elastic MapReduce)](#94-amazon-emr-elastic-mapreduce) | 🔴 Advanced |
| 9.5 | [Amazon Athena](#95-amazon-athena) | 🟡 Intermediate |
| 9.6 | [Amazon Redshift](#96-amazon-redshift) | 🔴 Advanced |
| 9.7 | [Amazon QuickSight → Amazon Quick Sight (part of AWS Quick)](#97-amazon-quicksight--amazon-quick-sight-part-of-aws-quick) | 🟡 Intermediate |

The sections follow the order data moves through a pipeline: **data-lake foundation → ingest → catalog & transform → query → visualise**.

## The modern AWS data-lake flow

```
Sources (apps, databases, IoT, logs)
   │
   │  streaming: Kinesis Data Streams → Data Firehose ... 9.2
   │  batch:     Glue jobs .............................. 9.3
   │             DMS (database replication) ............. see 6.8
   ▼
S3 data lake (raw → clean → curated zones) .............. 9.1
   │
   ├─ Glue Data Catalog — table schemas ................. 9.3
   ├─ Glue / EMR — transform with Spark ................. 9.3 / 9.4
   ├─ Athena — ad-hoc SQL directly on S3 ................ 9.5
   ├─ Redshift — data warehouse for BI at scale ......... 9.6
   ▼
Quick Sight — dashboards ................................ 9.7
SageMaker AI — machine learning ......................... see 7.2
```

Numbers on the right are the sections in this file; *see 6.8* and *see 7.2* point to [DMS](06-databases.md#68-aws-dms-database-migration-service) and [SageMaker AI](07-ai-ml.md#72-amazon-sagemaker-ai).

[↑ Back to contents](#contents)

---

## 9.1 Amazon S3 as a data lake

> **Level:** 🟡 Intermediate

See [05-storage-and-caching.md](05-storage-and-caching.md) for S3 basics. Data-lake specifics:
- Store in columnar formats (**Parquet**, ORC) and **partition** by date/region to cut scan costs
- Use open table formats — **Apache Iceberg** (or Hudi/Delta) — for ACID updates, time travel and schema evolution; **S3 Tables** provides managed Iceberg tables
- Organise into zones: `raw/`, `clean/`, `curated/`
- Govern access with **AWS Lake Formation**

[↑ Back to contents](#contents)

---

## 9.2 Amazon Kinesis

> **Level:** 🔴 Advanced

**What it is:** A family of real-time streaming services.

| Service | Purpose |
|---------|---------|
| **Kinesis Data Streams** | Durable, ordered stream with shards; custom consumers (Lambda, KCL apps, Flink); retention 24 h – 365 days |
| **Amazon Data Firehose** (formerly Kinesis Data Firehose) | Zero-code delivery of streams into S3, Redshift, OpenSearch, Splunk, Iceberg tables, with buffering and transformation |
| **Managed Service for Apache Flink** (formerly Kinesis Data Analytics) | Stateful real-time stream processing |
| **Kinesis Video Streams** | Ingest and process video streams |

**Use it for:** clickstreams, IoT telemetry, real-time dashboards, log ingestion, fraud detection.

**Python — put a record**
```python
import boto3, json
kinesis = boto3.client("kinesis")
kinesis.put_record(StreamName="clicks", Data=json.dumps({"user": 42, "page": "/pricing"}), PartitionKey="42")
```

**Pricing:** Data Streams per shard-hour (provisioned) or per GB in/out (on-demand); Firehose per GB ingested.

**Alternative:** **Amazon MSK** (managed Apache Kafka) when you need the Kafka ecosystem.

[↑ Back to contents](#contents)

---

## 9.3 AWS Glue

> **Level:** 🟡 Intermediate

**What it is:** Serverless data integration (ETL) service plus the central metadata catalog.

**Components**
- **Data Catalog**: Hive-compatible metastore used by Athena, EMR, Redshift Spectrum
- **Crawlers**: infer schemas from S3/DBs and populate the catalog
- **ETL jobs**: Spark (PySpark/Scala), Python shell, Ray; **Glue Studio** visual editor
- **Glue DataBrew**: no-code data preparation
- **Data Quality** rules, **job bookmarks** (incremental processing), workflows/triggers
- Glue Schema Registry for streaming schemas

**Pricing:** per DPU-hour for jobs/crawlers + catalog storage/requests beyond free tier.

[↑ Back to contents](#contents)

---

## 9.4 Amazon EMR (Elastic MapReduce)

> **Level:** 🔴 Advanced

**What it is:** Managed big-data frameworks — **Apache Spark**, Hive, Presto/Trino, Flink, HBase, Hudi/Iceberg.

**Deployment options**
- **EMR on EC2**: full control, Spot instances for task nodes (primary / core / task node roles)
- **EMR Serverless**: submit Spark/Hive jobs without managing clusters
- **EMR on EKS**: run Spark on your Kubernetes cluster
- EMR Studio notebooks

**Use it for:** large-scale ETL, ML feature engineering, log processing, petabyte joins.

**Pricing:** EMR fee per instance-hour on top of EC2 cost, or per vCPU/GB-hour for Serverless.

[↑ Back to contents](#contents)

---

## 9.5 Amazon Athena

> **Level:** 🟡 Intermediate

**What it is:** Serverless interactive SQL (Trino/Presto engine) directly on data in S3.

**Key concepts**
- Uses the **Glue Data Catalog** for table definitions
- **Federated queries** to other sources (RDS, DynamoDB, etc.) via connectors
- Supports Iceberg tables (`UPDATE`, `MERGE`, time travel)
- Workgroups for cost controls; **Athena for Apache Spark** notebooks
- Query results written to S3

```sql
SELECT region, SUM(amount) AS revenue
FROM sales_db.orders
WHERE dt BETWEEN '2026-11-01' AND '2026-11-22'
GROUP BY region
ORDER BY revenue DESC;
```

**Pricing:** per TB of data scanned (or provisioned capacity). Partitioning + Parquet can cut cost by 90%+.

**Python (boto3) — run a query and read the results**
```python
import time
import boto3

athena = boto3.client("athena")
qid = athena.start_query_execution(
    QueryString="SELECT region, SUM(amount) AS revenue FROM orders GROUP BY region",
    QueryExecutionContext={"Database": "sales_db"},
    WorkGroup="primary",   # the workgroup defines where results are saved in S3
)["QueryExecutionId"]

while True:
    state = athena.get_query_execution(QueryExecutionId=qid)["QueryExecution"]["Status"]["State"]
    if state in ("SUCCEEDED", "FAILED", "CANCELLED"):
        break
    time.sleep(1)

rows = athena.get_query_results(QueryExecutionId=qid)["ResultSet"]["Rows"]
for row in rows[1:]:                                  # row 0 is the header
    print([col.get("VarCharValue") for col in row["Data"]])
```

[↑ Back to contents](#contents)

---

## 9.6 Amazon Redshift

> **Level:** 🔴 Advanced

**What it is:** Managed, columnar, massively parallel **data warehouse** for SQL analytics.

**Key concepts**
- **Provisioned** (RA3 nodes with managed storage) or **Redshift Serverless** (RPUs, auto-scale)
- Distribution styles and sort keys (largely automatic now)
- **Redshift Spectrum**: query S3 data lake tables directly
- **Zero-ETL integrations** from Aurora, RDS, DynamoDB and some SaaS apps
- **Data sharing** across clusters/accounts, materialized views, concurrency scaling
- **Redshift ML**: create models with SQL (via SageMaker)

**Athena vs Redshift:** Athena for ad-hoc, infrequent queries on S3 with no infra; Redshift for frequent, complex, high-concurrency BI workloads with consistent performance.

**Pricing:** node-hours (provisioned) or RPU-hours (serverless) + managed storage.

[↑ Back to contents](#contents)

---

## 9.7 Amazon QuickSight → Amazon Quick Sight (part of AWS Quick)

> **Level:** 🟡 Intermediate

**What it is:** Serverless business-intelligence dashboards and reporting.

**Name change:** In October 2025 QuickSight became part of **Amazon Quick Suite** (an agentic AI workspace), with the BI part continuing as **Amazon Quick Sight**. The suite has since been shortened to Amazon Quick and, more recently, AWS Quick. Existing QuickSight APIs, SDKs and integrations continue to work.

**Key concepts**
- Data sources: Athena, Redshift, RDS/Aurora, S3, Snowflake, SaaS, files
- **SPICE**: in-memory engine for fast dashboards
- Analyses → published **dashboards**, paginated reports, scheduled email reports
- Natural-language questions and generative-AI authoring (formerly "Amazon Q in QuickSight")
- **Embedded analytics** in your own app; row-level security

**Pricing:** per user (author/reader roles) + SPICE capacity; capacity pricing for embedded use.

[↑ Back to contents](#contents)

---

[← 08 Application Integration (Coordination)](08-application-integration.md) · [🏠 README](../README.md) · [10 Monitoring & Logging →](10-monitoring-and-logging.md)
