# 06 — Databases

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| — | [Which database?](#which-database) | Overview |
| 6.1 | [Amazon RDS (Relational Database Service)](#61-amazon-rds-relational-database-service) | 🟢 Beginner |
| 6.2 | [Amazon Aurora](#62-amazon-aurora) | 🟡 Intermediate |
| 6.3 | [Amazon DynamoDB](#63-amazon-dynamodb) | 🟢 Beginner |
| 6.4 | [Amazon DocumentDB (with MongoDB compatibility)](#64-amazon-documentdb-with-mongodb-compatibility) | 🟡 Intermediate |
| 6.5 | [Amazon Keyspaces (for Apache Cassandra)](#65-amazon-keyspaces-for-apache-cassandra) | 🔴 Advanced |
| 6.6 | [Amazon Neptune](#66-amazon-neptune) | 🔴 Advanced |
| 6.7 | [Amazon OpenSearch Service](#67-amazon-opensearch-service) | 🔴 Advanced |
| 6.8 | [AWS DMS (Database Migration Service)](#68-aws-dms-database-migration-service) | 🔴 Advanced |

## Which database?

| Data / access pattern | Pick |
|-----------------------|------|
| Relational, SQL, joins, transactions | **RDS** (PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, Db2) or **Aurora** |
| Key-value / known access patterns, massive scale, serverless | **DynamoDB** |
| JSON documents, MongoDB API | **DocumentDB** |
| Wide-column, Cassandra (CQL) API | **Keyspaces** |
| Highly connected data (graphs) | **Neptune** |
| Full-text search, log analytics, vector search | **OpenSearch** |
| Moving data between databases | **DMS** |

[↑ Back to contents](#contents)

---

## 6.1 Amazon RDS (Relational Database Service)

> **Level:** 🟢 Beginner

**What it is:** Managed relational databases — AWS handles provisioning, patching, backups and failover.

**Engines:** PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, Db2.

**Key concepts**
- **Multi-AZ**: synchronous standby for high availability (Multi-AZ DB cluster adds 2 readable standbys)
- **Read replicas**: async copies for read scaling (also cross-region)
- **Automated backups** (point-in-time restore, up to 35 days) + manual snapshots
- **Parameter groups**, option groups
- **RDS Proxy**: connection pooling — very useful with Lambda
- IAM database authentication, Secrets Manager rotation, KMS encryption
- Blue/green deployments for low-downtime upgrades

**Pricing:** per instance-hour (or Reserved) + storage + I/O (by storage type) + backups beyond free allowance.

**Gotchas:** Put it in private subnets; enable deletion protection; size `max_connections` (or use RDS Proxy).

**Python (boto3) — connect to RDS PostgreSQL using a password from Secrets Manager**
```python
# pip install boto3 "psycopg[binary]"
import json
import boto3
import psycopg

secret = json.loads(
    boto3.client("secretsmanager").get_secret_value(SecretId="prod/db")["SecretString"]
)

with psycopg.connect(
    host=secret["host"], port=secret["port"], dbname=secret.get("dbname", "postgres"),
    user=secret["username"], password=secret["password"], sslmode="require",
) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT version();")
        print(cur.fetchone()[0])
```

[↑ Back to contents](#contents)

---

## 6.2 Amazon Aurora

> **Level:** 🟡 Intermediate

**What it is:** AWS's cloud-native relational database, compatible with **MySQL** and **PostgreSQL**, with a distributed storage layer.

**Key concepts**
- Storage replicated **6 ways across 3 AZs**, auto-grows up to 128 TiB (higher on some configs)
- Up to 15 low-lag **Aurora Replicas**; fast failover
- **Aurora Serverless v2**: scales capacity in fine-grained ACUs (can scale to zero)
- **Global Database**: cross-region replication, typically under a second
- **Aurora DSQL**: distributed, serverless PostgreSQL-compatible option for active-active multi-region
- Fast cloning, Backtrack (MySQL), zero-ETL integration into Redshift
- **pgvector** support for embeddings (Aurora PostgreSQL)

**Pricing:** instance-hours or ACU-hours + storage + I/O (or **I/O-Optimized** configuration with no I/O charges).

**RDS vs Aurora:** Aurora gives better availability/performance and features; RDS is cheaper for small or dev workloads and supports more engines.

[↑ Back to contents](#contents)

---

## 6.3 Amazon DynamoDB

> **Level:** 🟢 Beginner

**What it is:** Serverless key-value and document NoSQL database with single-digit-millisecond performance at any scale.

**Key concepts**
- **Table**, **item** (max 400 KB), **attributes**
- **Primary key**: partition key, or partition key + sort key
- **GSI** (Global Secondary Index) and **LSI** (Local Secondary Index)
- Capacity modes: **On-Demand** (pay per request) or **Provisioned** (+ auto scaling)
- **Streams** → Lambda for change-data-capture
- **TTL** for automatic expiry; **transactions**; **PITR** backups
- **Global Tables** for multi-region active-active
- **DAX**: in-memory cache for microsecond reads
- **Single-table design**: model access patterns first, then keys

**Python (boto3)**
```python
import boto3
from boto3.dynamodb.conditions import Key

table = boto3.resource("dynamodb").Table("Orders")
table.put_item(Item={"pk": "CUSTOMER#42", "sk": "ORDER#2026-11-23#001", "total": 499})

resp = table.query(
    KeyConditionExpression=Key("pk").eq("CUSTOMER#42") & Key("sk").begins_with("ORDER#2026-11")
)
orders = resp["Items"]
```

**Pricing:** On-Demand per read/write request unit, or provisioned RCU/WCU-hours; + storage; extra for streams reads, global tables, backups.

**Gotchas:** Avoid `Scan` in hot paths; design keys to avoid hot partitions; ad-hoc querying is hard — export to S3 + Athena for analytics.

[↑ Back to contents](#contents)

---

## 6.4 Amazon DocumentDB (with MongoDB compatibility)

> **Level:** 🟡 Intermediate

**What it is:** Managed JSON document database that supports the MongoDB API, drivers and tools.

**Key concepts:** Aurora-like architecture (separate compute and replicated storage), up to 15 replicas, elastic clusters for sharding, global clusters, vector search.

**Use it for:** apps already written against MongoDB, content catalogs, user profiles.

**Pricing:** instance-hours + storage + I/O (or I/O-optimized).

**Gotchas:** Not every MongoDB feature/operator is supported — check compatibility. Alternative: MongoDB Atlas on AWS.

[↑ Back to contents](#contents)

---

## 6.5 Amazon Keyspaces (for Apache Cassandra)

> **Level:** 🔴 Advanced

**What it is:** Serverless, Cassandra-compatible wide-column database — use CQL and existing Cassandra drivers.

**Use it for:** high-write workloads such as IoT telemetry, time-series events, migrating off self-managed Cassandra clusters.

**Key concepts:** keyspaces, tables, partition and clustering keys; on-demand or provisioned capacity; multi-region replication; PITR.

**Pricing:** per read/write request unit (or provisioned capacity) + storage.

[↑ Back to contents](#contents)

---

## 6.6 Amazon Neptune

> **Level:** 🔴 Advanced

**What it is:** Managed graph database.

**Query languages:** Gremlin (property graph), openCypher, SPARQL (RDF).

**Use it for:** social networks, recommendation engines, fraud detection rings, knowledge graphs, identity graphs, **GraphRAG** for generative AI.

**Variants:** Neptune Database (clusters, serverless option) and **Neptune Analytics** (in-memory graph analytics with vector search).

**Pricing:** instance-hours or Neptune Capacity Units + storage + I/O.

[↑ Back to contents](#contents)

---

## 6.7 Amazon OpenSearch Service

> **Level:** 🔴 Advanced

**What it is:** Managed OpenSearch (open-source fork of Elasticsearch) plus OpenSearch Dashboards.

**Use it for**
- Full-text search (product search, site search)
- Log and observability analytics
- **Vector search** for semantic search and RAG (k-NN)

**Key concepts:** domains (managed clusters) or **OpenSearch Serverless** collections; indices, shards, replicas; UltraWarm and cold storage tiers; ingestion via **OpenSearch Ingestion** pipelines; zero-ETL integrations (e.g. from DynamoDB, S3).

**Pricing:** instance-hours + storage (domains) or OpenSearch Compute Units (Serverless).

**Gotcha:** Not a primary database — keep the source of truth elsewhere and index into it.

[↑ Back to contents](#contents)

---

## 6.8 AWS DMS (Database Migration Service)

> **Level:** 🔴 Advanced

**What it is:** Migrates and replicates data between databases with minimal downtime.

**Key concepts**
- Source endpoint → **replication instance** (or **DMS Serverless**) → target endpoint
- Modes: **full load**, **full load + CDC** (ongoing change data capture), **CDC only**
- **Homogeneous** (Postgres → Postgres) or **heterogeneous** (Oracle → Aurora PostgreSQL)
- **DMS Schema Conversion** (and the older Schema Conversion Tool) converts schemas/code for heterogeneous moves
- Targets include S3, Kinesis, Redshift, DynamoDB, OpenSearch

**Pricing:** replication instance-hours (or DMS Capacity Units for Serverless) + storage.

**Gotchas:** Validate data after migration (DMS data validation); LOBs and unsupported data types need special settings.

[↑ Back to contents](#contents)

---

[← 05 Storage & Caching](05-storage-and-caching.md) · [🏠 README](../README.md) · [07 AI & Machine Learning →](07-ai-ml.md)
