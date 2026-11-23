# 13 — Glossary (plain language)

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

Terms you'll see throughout this library, explained for beginners. Grouped by theme, alphabetical within each group.

## Contents

Jump to: [Where things run](#where-things-run) · [Accounts, identity & security](#accounts-identity--security) · [Networking](#networking) · [Compute & containers](#compute--containers) · [Data & storage](#data--storage) · [Messaging & events](#messaging--events) · [Operations & DevOps](#operations--devops) · [AI](#ai)

---

## Where things run

| Term | Meaning |
|------|---------|
| **Availability Zone (AZ)** | One or more separate data centers inside a region (e.g. `eu-north-1a`). Running in 2+ AZs means one data center failing doesn't take your app down. |
| **Edge location** | A small AWS site close to users, used by CloudFront and Route 53 to deliver content faster. There are hundreds worldwide. |
| **Region** | A geographic area with its own AWS data centers, e.g. `eu-north-1` (Stockholm) or `us-east-1` (N. Virginia). Most resources live in one region; prices and available services differ between regions. |
| **Multi-AZ** | Running copies of something in more than one Availability Zone for high availability. |

[↑ Back to contents](#contents)

## Accounts, identity & security

| Term | Meaning |
|------|---------|
| **Access key** | A long-lived ID + secret used by scripts/CLI to call AWS. Avoid them where possible; prefer roles and temporary credentials. Never commit them to Git. |
| **ARN (Amazon Resource Name)** | The unique "full address" of any AWS resource, e.g. `arn:aws:s3:::my-bucket`. Used in policies. |
| **Encryption at rest / in transit** | At rest = data encrypted while stored on disk. In transit = encrypted while moving over the network (HTTPS/TLS). |
| **IAM policy** | A JSON document listing which actions are allowed or denied on which resources. |
| **IAM role** | An identity with permissions that a service, app or person *temporarily* assumes. Roles are the safe way to give your code AWS access. |
| **KMS (Key Management Service)** | Manages the encryption keys that many services use to encrypt your data. |
| **Least privilege** | Give only the permissions that are actually needed, nothing more. |
| **MFA** | Multi-factor authentication — a second step (app code, security key) on top of a password. |
| **Root user** | The email login that created the AWS account. It can do everything, so protect it with MFA and don't use it day-to-day. |
| **Security group** | A virtual firewall around a resource (EC2, RDS, Lambda in a VPC) that says which traffic may come in or go out. |

[↑ Back to contents](#contents)

## Networking

| Term | Meaning |
|------|---------|
| **CIDR block** | A way of writing an IP address range, e.g. `10.0.0.0/16` = 65,536 addresses. |
| **CDN** | Content Delivery Network — caches your content near users (CloudFront). |
| **DNS** | The internet's phone book: turns `example.com` into an IP address (Route 53). |
| **Internet Gateway** | Connects a VPC to the internet. |
| **Load balancer** | Spreads incoming traffic across several servers or containers. |
| **NAT Gateway** | Lets resources in a private subnet reach the internet (e.g. to download packages) without being reachable from it. Costs money per hour and per GB. |
| **Private subnet** | A subnet with no direct route from the internet — where databases and app servers should live. |
| **Public subnet** | A subnet with a route to the internet — for load balancers and NAT Gateways. |
| **Subnet** | A slice of a VPC's IP range, located in one Availability Zone. |
| **TLS / SSL certificate** | What makes `https://` work; proves the site's identity and encrypts traffic (ACM gives them free). |
| **VPC** | Virtual Private Cloud — your own private network inside AWS. |

[↑ Back to contents](#contents)

## Compute & containers

| Term | Meaning |
|------|---------|
| **AMI** | Amazon Machine Image — a template (OS + software) used to launch EC2 servers. |
| **Auto Scaling** | Automatically adds or removes servers/containers based on load. |
| **Cold start** | The extra delay when Lambda starts a new environment for your function after it has been idle. |
| **Container** | A packaged app plus everything it needs to run (usually built with Docker), so it runs the same everywhere. |
| **Container image / ECR** | The built, shippable version of a container. ECR (Elastic Container Registry) is where AWS stores them. |
| **Instance** | One virtual server in EC2. Instance *type* (e.g. `t3.micro`) sets its CPU and memory. |
| **Kubernetes** | Open-source system for running containers at scale (EKS is AWS's managed version). |
| **Orchestrator** | Software that starts, stops, scales and heals containers (ECS, Kubernetes). |
| **Serverless** | You don't manage servers; AWS runs and scales it, and you usually pay per use (Lambda, Fargate, DynamoDB, S3). There are still servers — you just don't see them. |
| **Spot instance** | Spare EC2 capacity at a big discount that AWS can take back with 2 minutes' notice. |

[↑ Back to contents](#contents)

## Data & storage

| Term | Meaning |
|------|---------|
| **Block storage** | A raw disk attached to one server (EBS). |
| **Bucket** | A top-level container for files (objects) in S3. Names are unique across all of AWS. |
| **Cache** | A fast, temporary copy of data kept in memory to avoid slow repeated lookups (ElastiCache). |
| **Data lake** | A central store (usually S3) holding raw and processed data in open file formats for analytics. |
| **Data warehouse** | A database optimised for analytics queries over large amounts of data (Redshift). |
| **ETL** | Extract, Transform, Load — moving data from sources, cleaning it, and loading it somewhere useful (Glue, EMR). |
| **File storage** | A shared folder many servers can use at once (EFS). |
| **NoSQL** | Databases that don't use fixed relational tables, e.g. key-value (DynamoDB) or document (DocumentDB). |
| **Object storage** | Stores files as objects with a key and metadata, accessed over HTTP (S3). |
| **Parquet** | A compressed, column-based file format that makes analytics queries much faster and cheaper. |
| **Partition key** | In DynamoDB, the attribute that decides where an item is stored — and how you look it up. |
| **Read replica** | A read-only copy of a database used to spread read traffic. |
| **Relational database (SQL)** | Tables with rows/columns and relationships, queried with SQL (RDS, Aurora). |
| **Snapshot** | A point-in-time backup of a disk or database. |

[↑ Back to contents](#contents)

## Messaging & events

| Term | Meaning |
|------|---------|
| **Asynchronous** | The caller doesn't wait for the work to finish — it hands the job off (e.g. to a queue) and moves on. |
| **Dead-letter queue (DLQ)** | Where messages go after failing too many times, so you can inspect them later. |
| **Decoupling** | Letting parts of a system communicate through queues/events instead of calling each other directly, so one can fail or scale without breaking the other. |
| **Event** | A record that something happened, e.g. "OrderPlaced" or "file uploaded". |
| **Event-driven** | An architecture where components react to events instead of being called directly. |
| **Idempotent** | Running the same operation twice has the same result as running it once — important because messages can be delivered more than once. |
| **Pub/sub** | Publish/subscribe: one sender publishes a message, many subscribers receive it (SNS). |
| **Queue** | A buffer where messages wait until a worker processes them (SQS). |

[↑ Back to contents](#contents)

## Operations & DevOps

| Term | Meaning |
|------|---------|
| **Alarm** | A rule that notifies you (or acts) when a metric crosses a threshold (CloudWatch). |
| **Blue/green deployment** | Run the new version alongside the old one, then switch traffic over — easy rollback. |
| **Canary deployment** | Send a small % of traffic to the new version first, then the rest if all is well. |
| **CI/CD** | Continuous Integration / Continuous Delivery — automatically build, test and deploy code on every change. |
| **Free tier** | AWS's limited free usage for new accounts and some always-free allowances. Check the current terms — not everything is free. |
| **High availability (HA)** | Designed to keep working when a component fails, usually by running in multiple AZs. |
| **Infrastructure as Code (IaC)** | Defining cloud resources in code/templates instead of clicking in the console (CloudFormation, CDK). |
| **Logs / metrics / traces** | The three pillars of observability: logs = text records, metrics = numbers over time, traces = a request's path through services. |
| **Managed service** | AWS handles servers, patching, backups and scaling; you focus on using it (e.g. RDS vs installing PostgreSQL on EC2 yourself). |
| **On-demand** | Pay by the second/hour/request with no commitment. |
| **Reserved Instances / Savings Plans** | Commit to 1–3 years of usage in exchange for a large discount. |
| **Shared Responsibility Model** | AWS secures the cloud itself (hardware, data centers); you secure what you put in it (data, IAM, configuration). |
| **SLA** | Service Level Agreement — AWS's published uptime commitment for a service. |

[↑ Back to contents](#contents)

## AI

| Term | Meaning |
|------|---------|
| **Embedding** | A list of numbers representing the meaning of text or an image, so similar things are close together. |
| **Fine-tuning** | Further training an existing model on your own data. |
| **Foundation model (FM) / LLM** | A large pre-trained AI model (e.g. Claude, Llama, Nova) you can use through Bedrock. |
| **Inference** | Using a trained model to make a prediction or generate output. |
| **MCP (Model Context Protocol)** | An open standard that lets AI assistants connect to tools and data sources — e.g. the AWS MCP Server. |
| **RAG** | Retrieval-Augmented Generation — fetch relevant documents first, then give them to the LLM so its answer is grounded in your data. |
| **Token** | A chunk of text (roughly ¾ of an English word) — LLM usage is priced per token. |
| **Vector database / vector search** | Stores embeddings and finds the most similar ones quickly (OpenSearch, Aurora pgvector, S3 Vectors). |

[↑ Back to contents](#contents)

---

[← 12 Reference Architectures & Cheat Sheet](12-reference-architectures.md) · [🏠 README](../README.md)
