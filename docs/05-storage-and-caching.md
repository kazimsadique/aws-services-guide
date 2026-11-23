# 05 — Storage & Caching

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 5.1 | [Amazon S3 (Simple Storage Service)](#51-amazon-s3-simple-storage-service) | 🟢 Beginner |
| 5.2 | [Amazon EBS (Elastic Block Store)](#52-amazon-ebs-elastic-block-store) | 🟢 Beginner |
| 5.3 | [Amazon EFS (Elastic File System)](#53-amazon-efs-elastic-file-system) | 🟡 Intermediate |
| 5.4 | [Amazon ElastiCache](#54-amazon-elasticache) | 🟡 Intermediate |
| 5.5 | [Amazon MemoryDB](#55-amazon-memorydb) | 🔴 Advanced |

| Need | Service |
|------|---------|
| Objects/files over HTTP, unlimited scale | **S3** |
| A disk for one EC2 instance | **EBS** |
| Shared POSIX file system for many instances/containers | **EFS** |
| In-memory cache in front of a DB | **ElastiCache** |
| Durable in-memory primary database (Redis/Valkey API) | **MemoryDB** |

---

## 5.1 Amazon S3 (Simple Storage Service)

> **Level:** 🟢 Beginner

**What it is:** Object storage — virtually unlimited, designed for 99.999999999% (11 nines) durability.

**Use it for:** static websites (with CloudFront), user uploads, backups, logs, data lakes, ML datasets, build artifacts.

**Key concepts**
- **Buckets** (globally unique names) and **objects** (key + data + metadata, up to 5 TB each)
- **Storage classes**: Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Express One Zone (single-digit ms), Glacier Instant / Flexible / Deep Archive
- **Lifecycle rules**: transition/expire objects automatically
- **Versioning**, **Object Lock** (WORM), **replication** (CRR/SRR)
- Security: **Block Public Access** (on by default), bucket policies, IAM, SSE-S3/SSE-KMS encryption (on by default)
- **Presigned URLs**: time-limited upload/download without exposing credentials
- **Event notifications** → Lambda / SQS / SNS / EventBridge
- Newer specialized buckets: **S3 Tables** (Apache Iceberg) and **S3 Vectors** (vector storage for AI)

**Python (boto3) — presigned upload URL**
```python
import boto3

s3 = boto3.client("s3")
url = s3.generate_presigned_url(
    "put_object",
    Params={"Bucket": "my-uploads", "Key": "user/123/avatar.png", "ContentType": "image/png"},
    ExpiresIn=900,
)
```

**Pricing:** per GB-month (by class) + requests + data transfer out + retrieval fees for IA/Glacier.

**Gotchas:** Never make buckets public to serve a website — use CloudFront + OAC. Many small objects in IA/Glacier classes cost more than expected (minimum object size/duration).

[↑ Back to contents](#contents)

---

## 5.2 Amazon EBS (Elastic Block Store)

> **Level:** 🟢 Beginner

**What it is:** Network-attached block storage (virtual disks) for EC2.

**Key concepts**
- Volume types: **gp3** (default general SSD, tune IOPS/throughput separately), **io2 Block Express** (high-IOPS databases), **st1/sc1** (throughput/cold HDD)
- Lives in **one AZ**; usually attached to one instance (Multi-Attach for io1/io2)
- **Snapshots** to S3 (incremental), copy across regions; **Data Lifecycle Manager** / AWS Backup for automation
- Encryption with KMS

**Pricing:** per provisioned GB-month (+ provisioned IOPS/throughput) + snapshot storage.

**Gotcha:** You pay for provisioned size even if empty; delete unattached volumes.

**Python (boto3) — back up a volume with a snapshot**
```python
import boto3

ec2 = boto3.client("ec2")
snap = ec2.create_snapshot(
    VolumeId="vol-0123456789abcdef0",
    Description="Before upgrading the app",
    TagSpecifications=[{"ResourceType": "snapshot",
                        "Tags": [{"Key": "Purpose", "Value": "pre-upgrade"}]}],
)
print(snap["SnapshotId"], snap["State"])   # "pending" → "completed"
```

[↑ Back to contents](#contents)

---

## 5.3 Amazon EFS (Elastic File System)

> **Level:** 🟡 Intermediate

**What it is:** Managed, elastic NFS file system shared across many EC2 instances, containers (ECS/EKS/Fargate) and Lambda.

**Key concepts**
- Regional (multi-AZ) or One Zone
- Storage classes: Standard, Infrequent Access, Archive + lifecycle policies
- Throughput modes: **Elastic** (recommended), Bursting, Provisioned
- **Access points** for app-specific entry directories and POSIX identities

**Use it for:** shared content (CMS uploads), home directories, ML model files shared by containers.

**Pricing:** per GB-month stored (by class) + throughput/access charges in Elastic mode.

**Related:** **Amazon FSx** for Windows File Server, Lustre (HPC), NetApp ONTAP, OpenZFS.

[↑ Back to contents](#contents)

---

## 5.4 Amazon ElastiCache

> **Level:** 🟡 Intermediate

**What it is:** Managed in-memory cache — **Valkey**, **Redis OSS** or **Memcached**.

**Use it for:** caching DB queries, session storage, rate limiting, leaderboards (sorted sets), pub/sub, queues.

**Key concepts**
- **Serverless** option (no node sizing) or node-based clusters
- Cluster mode (sharding), replicas, Multi-AZ automatic failover
- Caching strategies: **cache-aside** (lazy loading), **write-through**, TTLs

**Python cache-aside example**
```python
import json, redis

r = redis.Redis(host="my-cache.xxxxxx.cache.amazonaws.com", port=6379, ssl=True)

def get_user(user_id, db):  # db = your own database access object
    key = f"user:{user_id}"
    if (cached := r.get(key)):
        return json.loads(cached)
    user = db.fetch_user(user_id)
    r.setex(key, 300, json.dumps(user))  # 5-minute TTL
    return user
```

**Pricing:** per node-hour, or for Serverless per GB-hour stored + ElastiCache Processing Units. Valkey is priced lower than Redis OSS.

**Gotcha:** ElastiCache is a cache — design for data loss; the source of truth stays in the database.

[↑ Back to contents](#contents)

---

## 5.5 Amazon MemoryDB

> **Level:** 🔴 Advanced

**What it is:** A **durable** in-memory database compatible with Valkey/Redis OSS — data is persisted in a Multi-AZ transaction log.

**Use it for:** when you want Redis-style data structures as your **primary database** (sessions, shopping carts, leaderboards, real-time features) with microsecond reads. Also supports **vector search**.

**ElastiCache vs MemoryDB**
| | ElastiCache | MemoryDB |
|-|-------------|----------|
| Role | Cache in front of a DB | Primary database |
| Durability | Can lose recent writes on failure | Durable, Multi-AZ log |
| Write latency | Microseconds | Single-digit milliseconds |
| Cost | Lower | Higher |

**Pricing:** per node-hour + data written (GB).

[↑ Back to contents](#contents)

---

[← 04 APIs & Compute](04-apis-and-compute.md) · [🏠 README](../README.md) · [06 Databases →](06-databases.md)
