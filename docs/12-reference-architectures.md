# 12 — Reference Architectures & Cheat Sheet

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

How the services from files 01–11 combine in real systems.

## Contents

| Section | Level |
|---------|-------|
| [A. Static website / SPA](#a-static-website--spa---beginner) | 🟢 Beginner |
| [B. Classic three-tier web app (containers)](#b-classic-three-tier-web-app-containers---intermediate) | 🟡 Intermediate |
| [C. Serverless REST API](#c-serverless-rest-api---beginner---intermediate) | 🟢 Beginner → 🟡 Intermediate |
| [D. Event-driven order processing](#d-event-driven-order-processing---intermediate) | 🟡 Intermediate |
| [E. GenAI / RAG application](#e-genai--rag-application---intermediate) | 🟡 Intermediate |
| [F. Analytics platform](#f-analytics-platform---advanced) | 🔴 Advanced |
| [Cheat sheet: "Which one do I pick?"](#cheat-sheet-which-one-do-i-pick) |  |
| [Suggested learning order (backend / Python focus)](#suggested-learning-order-backend--python-focus) |  |

---

Architectures are ordered from 🟢 beginner (A) to 🔴 advanced (F).

## A. Static website / SPA — 🟢 Beginner

```
Route 53 → CloudFront (+ WAF, ACM cert in us-east-1) → S3 (private, OAC)
```
Optional: Amplify Hosting does all of this for you with Git-based deploys.

[↑ Back to contents](#contents)

---

## B. Classic three-tier web app (containers) — 🟡 Intermediate

```
Route 53 → CloudFront (+ WAF) → ALB (public subnets)
                               │
                               ▼
                    ECS on Fargate (private subnets, 2+ AZs)
                      │            │             │
                      ▼            ▼             ▼
            Aurora PostgreSQL  ElastiCache     S3 (uploads)
            (isolated subnets)  (Valkey)
Secrets Manager for DB creds · CloudWatch logs/alarms · CodePipeline → CodeBuild → ECR → ECS deploy
Infrastructure defined in CDK (Python)
```

[↑ Back to contents](#contents)

---

## C. Serverless REST API — 🟢 Beginner → 🟡 Intermediate

```
Route 53 → API Gateway (HTTP API, Cognito JWT authorizer)
             → Lambda (Python + Powertools)
                → DynamoDB (single-table design)
                → EventBridge (domain events) → SQS → Lambda workers
Step Functions for multi-step business processes
```
Scales to zero; pay per request.

[↑ Back to contents](#contents)

---

## D. Event-driven order processing — 🟡 Intermediate

```
Order API → EventBridge bus "orders"
   ├─ rule: OrderPlaced → Step Functions (reserve stock → charge payment → ship)
   ├─ rule: OrderPlaced → SQS → email/notification service (SNS)
   └─ rule: all events  → Firehose → S3 data lake
```

[↑ Back to contents](#contents)

---

## E. GenAI / RAG application — 🟡 Intermediate

```
Documents in S3 → Bedrock Knowledge Base (chunk + embed)
                     → vector store (OpenSearch Serverless / Aurora pgvector / S3 Vectors)
User → API Gateway → Lambda/ECS (FastAPI)
        → Bedrock (Converse API / Agents, Guardrails)
        → answer with citations
Optional: Transcribe (voice in) · Polly (voice out) · DynamoDB (chat history)
```

[↑ Back to contents](#contents)

---

## F. Analytics platform — 🔴 Advanced

```
App DBs ──DMS (CDC)──► S3 raw ──Glue/EMR Spark──► S3 curated (Iceberg)
Clickstream ─Kinesis─► Firehose ─► S3 raw          │
                                                   ├─► Athena (ad-hoc SQL)
                                                   ├─► Redshift (warehouse, BI)
                                                   └─► SageMaker (ML)
Orchestration: MWAA (Airflow) or Step Functions · Dashboards: Quick Sight
```

[↑ Back to contents](#contents)

---

## Cheat sheet: "Which one do I pick?"

| Decision | Option A | Option B | Rule of thumb |
|----------|----------|----------|---------------|
| Containers | ECS | EKS | ECS unless you need Kubernetes |
| Container capacity | Fargate | EC2 | Fargate unless you need GPUs, host access or max cost efficiency at scale |
| Functions vs containers | Lambda | Fargate | Lambda for < 15 min event-driven work; Fargate for long-running services |
| API front door | API Gateway | ALB | API GW for serverless/API mgmt; ALB for steady container traffic |
| REST vs GraphQL | API Gateway | AppSync | GraphQL/real-time → AppSync |
| Relational | RDS | Aurora | Aurora for production HA/scale; RDS for small/cheap or other engines |
| SQL vs NoSQL | Aurora/RDS | DynamoDB | DynamoDB when access patterns are known and scale is huge |
| Cache vs in-memory DB | ElastiCache | MemoryDB | Need durability → MemoryDB |
| Block vs file storage | EBS | EFS | Shared across instances → EFS |
| Queue vs pub/sub | SQS | SNS | Combine: SNS → many SQS |
| Bus vs topic | EventBridge | SNS | Content routing/SaaS/replay → EventBridge |
| Workflow | Step Functions | MWAA | App workflows → Step Functions; scheduled data DAGs → Airflow |
| Query S3 vs warehouse | Athena | Redshift | Ad-hoc → Athena; heavy BI → Redshift |
| ETL | Glue | EMR | Glue serverless & simpler; EMR for control/big Spark |
| Secrets vs config | Secrets Manager | Parameter Store / AppConfig | Rotation → Secrets Manager; flags → AppConfig |
| IaC | CloudFormation | CDK | CDK if you prefer real code (it generates CloudFormation) |
| LLMs vs custom ML | Bedrock | SageMaker AI | Use Bedrock first; SageMaker for training your own |
| Human sign-in to AWS | IAM users | Identity Center | Identity Center |
| App end-user sign-in | — | Cognito | Cognito (or a third-party IdP) |

[↑ Back to contents](#contents)

---

## Suggested learning order (backend / Python focus)

1. IAM, VPC basics, S3, EC2 (foundations)
2. Lambda + API Gateway + DynamoDB (serverless API)
3. RDS/Aurora + ElastiCache + Secrets Manager
4. ECS on Fargate + ALB + ECR
5. SQS, SNS, EventBridge, Step Functions
6. CloudWatch, CloudTrail, X-Ray/OpenTelemetry
7. CDK (Python) + CodePipeline / GitHub Actions with OIDC
8. Bedrock (RAG, agents) and the AWS MCP Server
9. Data: Glue, Athena, Kinesis, Redshift, MWAA

Certifications that map to this:
- 🟢 **AWS Certified Cloud Practitioner** — the best first exam for beginners
- 🟡 **Developer – Associate** or **Solutions Architect – Associate**
- 🔴 **Solutions Architect – Professional**, or a specialty track (e.g. Machine Learning Engineer – Associate or the generative-AI certification for the AI track)

[↑ Back to contents](#contents)

---

[← 11 Developer Productivity](11-developer-productivity.md) · [🏠 README](../README.md) · [13 Glossary (plain language) →](13-glossary.md)
