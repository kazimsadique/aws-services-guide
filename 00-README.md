# AWS Services Guide: From Beginner to Architect

**A free, open guide to the AWS services that actually matter — for beginners, intermediate and advanced learners.**

## 👋 Why this repo exists

AWS offers **more than 240 services**. When you open the AWS console for the first time, that number is overwhelming: Which ones do real companies use? Where should you start? What's the difference between SQS, SNS and EventBridge — and do you even need them?

Most of those 240+ services are specialised tools that you may never touch. In practice, **around 60 services make up the vast majority of real-world cloud systems** — the websites, APIs, mobile backends, data platforms and AI applications you use every day.

This repository gathers those ~60 services in one place, explains each one in plain language, and shows how they fit together, so you can spend your time **building** instead of searching.

## 🙋 Who it's for

| If you are… | This repo helps you… |
|-------------|----------------------|
| 🟢 **A beginner** — new to AWS or the cloud | Know which few services to learn first, understand the jargon (see the [glossary](13-glossary.md)), and avoid surprise bills |
| 🟡 **An intermediate developer** — you've deployed something on AWS | Choose between similar services, design production-ready architectures, and prepare for Associate-level certifications |
| 🔴 **An advanced engineer or architect** — you design larger systems | Get a quick refresher on specialised services, compare trade-offs at a glance, and catch up on recent AWS changes |

Whatever your level, if you're curious about AWS, you're welcome here.

## 📚 What you'll find

- **~60 services in 11 categories** — networking, security, compute, databases, AI, data analytics and more
- **The same layout for every service** — what it is, when to use it, key concepts, pricing and common mistakes
- **Python (boto3) examples for 28 services**, including every 🟢 beginner service you'd call from code — see [how to run them](#-running-the-python-examples)
- **A level badge** (🟢 🟡 🔴) on every service, so you know what to learn now and what can wait
- **"Which one do I pick?" comparisons** and **six reference architectures**, from a static website to a GenAI/RAG app
- **Up-to-date notes** on recent changes, such as Amazon Q Developer → Kiro, the AWS MCP Server, X-Ray → OpenTelemetry and QuickSight → Quick Sight
- **A plain-language glossary** of ~70 cloud terms

## 🧭 How to use it

1. Find your level in **[🎯 Pick your level](#-pick-your-level)** below — or browse by topic in **[📂 Services by category](#-services-by-category)**.
2. New to AWS? Follow **[🌱 Start here](#-start-here-new-to-aws)** step by step.
3. Use the contents table at the top of each file to jump to a specific service.

> Last reviewed: November 2026. AWS renames and retires services regularly — check the official AWS documentation before relying on anything here in production. Spotted something outdated? Corrections and pull requests are very welcome.

## 🎯 Pick your level

Every service in this library is tagged with a level badge right under its heading. Find your level below and follow that path.

| Level | Who it's for | Services | Goal |
|-------|--------------|----------|------|
| 🟢 **Beginner** | New to AWS or cloud; can write basic code | **Start with:** [IAM](02-identity-security-config.md#21-aws-iam-identity-and-access-management), [S3](05-storage-and-caching.md#51-amazon-s3-simple-storage-service), [EC2](04-apis-and-compute.md#43-amazon-ec2-elastic-compute-cloud), [Lambda](04-apis-and-compute.md#48-aws-lambda), [DynamoDB](06-databases.md#63-amazon-dynamodb), [RDS](06-databases.md#61-amazon-rds-relational-database-service), [CloudWatch](10-monitoring-and-logging.md#101-amazon-cloudwatch)<br>**Then:** [API Gateway](04-apis-and-compute.md#42-amazon-api-gateway), [CloudFront](01-networking-and-edge.md#12-amazon-cloudfront), [Certificate Manager](01-networking-and-edge.md#17-aws-certificate-manager-acm), [EBS](05-storage-and-caching.md#52-amazon-ebs-elastic-block-store), [Lightsail](04-apis-and-compute.md#44-amazon-lightsail), [SQS](08-application-integration.md#82-amazon-sqs-simple-queue-service), [SNS](08-application-integration.md#81-amazon-sns-simple-notification-service), [CloudTrail](10-monitoring-and-logging.md#102-aws-cloudtrail), [IAM Identity Center](02-identity-security-config.md#22-aws-iam-identity-center-successor-to-aws-sso), [Bedrock](07-ai-ml.md#71-amazon-bedrock), [Rekognition](07-ai-ml.md#73-amazon-rekognition), [Polly](07-ai-ml.md#74-amazon-polly), [Transcribe](07-ai-ml.md#75-amazon-transcribe), [Amplify](11-developer-productivity.md#111-aws-amplify), [WorkSpaces](11-developer-productivity.md#113-amazon-workspaces), [Amazon Q / Kiro](11-developer-productivity.md#114-amazon-q--kiro-ai-assistants) | Deploy a small app; understand the core building blocks; pass **AWS Certified Cloud Practitioner** |
| 🟡 **Intermediate** | Built something on AWS; ready for production-style systems | [VPC](01-networking-and-edge.md#15-amazon-vpc-virtual-private-cloud), [Route 53](01-networking-and-edge.md#11-amazon-route-53), [WAF](01-networking-and-edge.md#13-aws-waf-web-application-firewall), [Shield](01-networking-and-edge.md#14-aws-shield), [Cognito](02-identity-security-config.md#23-amazon-cognito), [Secrets Manager](02-identity-security-config.md#24-aws-secrets-manager), [CloudFormation](03-iac-and-cicd.md#31-aws-cloudformation), [CDK](03-iac-and-cicd.md#32-aws-cdk-cloud-development-kit), [CodeBuild](03-iac-and-cicd.md#33-aws-codebuild), [CodePipeline](03-iac-and-cicd.md#35-aws-codepipeline), [Load Balancers](04-apis-and-compute.md#41-elastic-load-balancing-elb), [ECS](04-apis-and-compute.md#45-amazon-ecs-elastic-container-service), [Fargate](04-apis-and-compute.md#46-aws-fargate), [EFS](05-storage-and-caching.md#53-amazon-efs-elastic-file-system), [ElastiCache](05-storage-and-caching.md#54-amazon-elasticache), [Aurora](06-databases.md#62-amazon-aurora), [DocumentDB](06-databases.md#64-amazon-documentdb-with-mongodb-compatibility), [EventBridge](08-application-integration.md#83-amazon-eventbridge), [Step Functions](08-application-integration.md#84-aws-step-functions), [S3 data lake](09-data-processing-analytics.md#91-amazon-s3-as-a-data-lake), [Athena](09-data-processing-analytics.md#95-amazon-athena), [Glue](09-data-processing-analytics.md#93-aws-glue), [Quick Sight](09-data-processing-analytics.md#97-amazon-quicksight--amazon-quick-sight-part-of-aws-quick), [AWS Config](10-monitoring-and-logging.md#103-aws-config), [X-Ray](10-monitoring-and-logging.md#104-aws-x-ray), [AppSync](11-developer-productivity.md#112-aws-appsync), [AWS MCP Server](11-developer-productivity.md#115-aws-mcp-server-model-context-protocol) | Secure networking, containers, IaC, CI/CD, event-driven design; **Developer** or **Solutions Architect – Associate** |
| 🔴 **Advanced** | Designing large, specialised or multi-team systems | [VPN](01-networking-and-edge.md#16-aws-vpn), [AppConfig](02-identity-security-config.md#25-aws-appconfig), [CodeDeploy](03-iac-and-cicd.md#34-aws-codedeploy), [EKS](04-apis-and-compute.md#47-amazon-eks-elastic-kubernetes-service), [MemoryDB](05-storage-and-caching.md#55-amazon-memorydb), [Keyspaces](06-databases.md#65-amazon-keyspaces-for-apache-cassandra), [Neptune](06-databases.md#66-amazon-neptune), [OpenSearch](06-databases.md#67-amazon-opensearch-service), [DMS](06-databases.md#68-aws-dms-database-migration-service), [SageMaker AI](07-ai-ml.md#72-amazon-sagemaker-ai), [MWAA (Airflow)](08-application-integration.md#85-amazon-mwaa-managed-workflows-for-apache-airflow), [EMR](09-data-processing-analytics.md#94-amazon-emr-elastic-mapreduce), [Redshift](09-data-processing-analytics.md#96-amazon-redshift), [Kinesis](09-data-processing-analytics.md#92-amazon-kinesis) | Kubernetes, big data, migrations, ML platforms; **Solutions Architect – Professional** or a specialty cert |

> The levels describe how much background a service needs, not how important it is. VPC is 🟡 but every real system uses it, so beginners should read its section once they're comfortable with the 🟢 path.

## 📂 Services by category

Prefer to browse by topic? Every service links straight to its section; the badge shows its level.

| # | Category | Services |
|---|----------|----------|
| 01 | [🌐 Networking & Edge](01-networking-and-edge.md) | [Route 53](01-networking-and-edge.md#11-amazon-route-53) 🟡 · [CloudFront](01-networking-and-edge.md#12-amazon-cloudfront) 🟢 · [WAF](01-networking-and-edge.md#13-aws-waf-web-application-firewall) 🟡 · [Shield](01-networking-and-edge.md#14-aws-shield) 🟡 · [VPC](01-networking-and-edge.md#15-amazon-vpc-virtual-private-cloud) 🟡 · [VPN](01-networking-and-edge.md#16-aws-vpn) 🔴 · [Certificate Manager](01-networking-and-edge.md#17-aws-certificate-manager-acm) 🟢 |
| 02 | [🔐 Identity, Security & Config](02-identity-security-config.md) | [IAM](02-identity-security-config.md#21-aws-iam-identity-and-access-management) 🟢 · [IAM Identity Center](02-identity-security-config.md#22-aws-iam-identity-center-successor-to-aws-sso) 🟢 · [Cognito](02-identity-security-config.md#23-amazon-cognito) 🟡 · [Secrets Manager](02-identity-security-config.md#24-aws-secrets-manager) 🟡 · [AppConfig](02-identity-security-config.md#25-aws-appconfig) 🔴 |
| 03 | [🏗️ Infrastructure as Code & CI/CD](03-iac-and-cicd.md) | [CloudFormation](03-iac-and-cicd.md#31-aws-cloudformation) 🟡 · [CDK](03-iac-and-cicd.md#32-aws-cdk-cloud-development-kit) 🟡 · [CodeBuild](03-iac-and-cicd.md#33-aws-codebuild) 🟡 · [CodeDeploy](03-iac-and-cicd.md#34-aws-codedeploy) 🔴 · [CodePipeline](03-iac-and-cicd.md#35-aws-codepipeline) 🟡 |
| 04 | [⚙️ APIs & Compute](04-apis-and-compute.md) | [Load Balancers](04-apis-and-compute.md#41-elastic-load-balancing-elb) 🟡 · [API Gateway](04-apis-and-compute.md#42-amazon-api-gateway) 🟢 · [EC2](04-apis-and-compute.md#43-amazon-ec2-elastic-compute-cloud) 🟢 · [Lightsail](04-apis-and-compute.md#44-amazon-lightsail) 🟢 · [ECS](04-apis-and-compute.md#45-amazon-ecs-elastic-container-service) 🟡 · [Fargate](04-apis-and-compute.md#46-aws-fargate) 🟡 · [EKS](04-apis-and-compute.md#47-amazon-eks-elastic-kubernetes-service) 🔴 · [Lambda](04-apis-and-compute.md#48-aws-lambda) 🟢 |
| 05 | [💾 Storage & Caching](05-storage-and-caching.md) | [S3](05-storage-and-caching.md#51-amazon-s3-simple-storage-service) 🟢 · [EBS](05-storage-and-caching.md#52-amazon-ebs-elastic-block-store) 🟢 · [EFS](05-storage-and-caching.md#53-amazon-efs-elastic-file-system) 🟡 · [ElastiCache](05-storage-and-caching.md#54-amazon-elasticache) 🟡 · [MemoryDB](05-storage-and-caching.md#55-amazon-memorydb) 🔴 |
| 06 | [🗄️ Databases](06-databases.md) | [RDS](06-databases.md#61-amazon-rds-relational-database-service) 🟢 · [Aurora](06-databases.md#62-amazon-aurora) 🟡 · [DynamoDB](06-databases.md#63-amazon-dynamodb) 🟢 · [DocumentDB](06-databases.md#64-amazon-documentdb-with-mongodb-compatibility) 🟡 · [Keyspaces](06-databases.md#65-amazon-keyspaces-for-apache-cassandra) 🔴 · [Neptune](06-databases.md#66-amazon-neptune) 🔴 · [OpenSearch](06-databases.md#67-amazon-opensearch-service) 🔴 · [DMS](06-databases.md#68-aws-dms-database-migration-service) 🔴 |
| 07 | [🤖 AI & Machine Learning](07-ai-ml.md) | [Bedrock](07-ai-ml.md#71-amazon-bedrock) 🟢 · [SageMaker AI](07-ai-ml.md#72-amazon-sagemaker-ai) 🔴 · [Rekognition](07-ai-ml.md#73-amazon-rekognition) 🟢 · [Polly](07-ai-ml.md#74-amazon-polly) 🟢 · [Transcribe](07-ai-ml.md#75-amazon-transcribe) 🟢 |
| 08 | [🔄 Application Integration](08-application-integration.md) | [SNS](08-application-integration.md#81-amazon-sns-simple-notification-service) 🟢 · [SQS](08-application-integration.md#82-amazon-sqs-simple-queue-service) 🟢 · [EventBridge](08-application-integration.md#83-amazon-eventbridge) 🟡 · [Step Functions](08-application-integration.md#84-aws-step-functions) 🟡 · [MWAA (Airflow)](08-application-integration.md#85-amazon-mwaa-managed-workflows-for-apache-airflow) 🔴 |
| 09 | [📊 Data Processing & Analytics](09-data-processing-analytics.md) | [S3 data lake](09-data-processing-analytics.md#91-amazon-s3-as-a-data-lake) 🟡 · [Kinesis](09-data-processing-analytics.md#92-amazon-kinesis) 🔴 · [Glue](09-data-processing-analytics.md#93-aws-glue) 🟡 · [EMR](09-data-processing-analytics.md#94-amazon-emr-elastic-mapreduce) 🔴 · [Athena](09-data-processing-analytics.md#95-amazon-athena) 🟡 · [Redshift](09-data-processing-analytics.md#96-amazon-redshift) 🔴 · [Quick Sight](09-data-processing-analytics.md#97-amazon-quicksight--amazon-quick-sight-part-of-aws-quick) 🟡 |
| 10 | [📈 Monitoring & Logging](10-monitoring-and-logging.md) | [CloudWatch](10-monitoring-and-logging.md#101-amazon-cloudwatch) 🟢 · [CloudTrail](10-monitoring-and-logging.md#102-aws-cloudtrail) 🟢 · [AWS Config](10-monitoring-and-logging.md#103-aws-config) 🟡 · [X-Ray](10-monitoring-and-logging.md#104-aws-x-ray) 🟡 |
| 11 | [🛠️ Developer Productivity](11-developer-productivity.md) | [Amplify](11-developer-productivity.md#111-aws-amplify) 🟢 · [AppSync](11-developer-productivity.md#112-aws-appsync) 🟡 · [WorkSpaces](11-developer-productivity.md#113-amazon-workspaces) 🟢 · [Amazon Q / Kiro](11-developer-productivity.md#114-amazon-q--kiro-ai-assistants) 🟢 · [AWS MCP Server](11-developer-productivity.md#115-aws-mcp-server-model-context-protocol) 🟡 |
| 12 | [🗺️ Reference Architectures](12-reference-architectures.md) | 6 architectures (🟢 → 🔴), a "which one do I pick?" cheat sheet, and a learning order |
| 13 | [📖 Glossary](13-glossary.md) | ~70 cloud terms in plain language, grouped by theme |

## 🌱 Start here (new to AWS?)

You do **not** need to learn 60 services. Almost every AWS system is built on a handful of core ones. Learn these first, in this order:

| Step | Service | Why first | Read |
|------|---------|-----------|------|
| 1 | **IAM** | Controls who can do what. Every other service depends on it. | [2.1 →](02-identity-security-config.md#21-aws-iam-identity-and-access-management) |
| 2 | **S3** | Stores files. The simplest service to try and used everywhere. | [5.1 →](05-storage-and-caching.md#51-amazon-s3-simple-storage-service) |
| 3 | **EC2** | A virtual server — the "classic" way to run code in the cloud. | [4.3 →](04-apis-and-compute.md#43-amazon-ec2-elastic-compute-cloud) |
| 4 | **Lambda** | Run code without managing servers. | [4.8 →](04-apis-and-compute.md#48-aws-lambda) |
| 5 | **DynamoDB** or **RDS** | Store your app's data (NoSQL or SQL). | [6.3 DynamoDB →](06-databases.md#63-amazon-dynamodb) · [6.1 RDS →](06-databases.md#61-amazon-rds-relational-database-service) |
| 6 | **CloudWatch** | See logs and errors when something breaks. | [10.1 →](10-monitoring-and-logging.md#101-amazon-cloudwatch) |

Then read [12-reference-architectures.md](12-reference-architectures.md) to see how they fit together. Everything else can wait until a project needs it.

**Before you start**
- 📖 Keep the [glossary](13-glossary.md) open — it explains terms like *region*, *subnet* and *ARN* in plain words.
- 💸 Create a **billing budget alert** (AWS Billing → Budgets) on day one, e.g. $5/month. Most beginner horror stories are surprise bills.
- 🔒 Turn on **MFA** for the root user, then stop using root and sign in with an IAM Identity Center user instead.
- 🧹 Delete what you create when you finish experimenting (EC2 instances, NAT Gateways, databases and load balancers cost money even when idle).

### 🐍 Running the Python examples

The examples use **boto3**, the official AWS SDK for Python. To run them:

```bash
# 1. Install the AWS CLI (https://aws.amazon.com/cli/) and Python 3.10+
# 2. Sign in — IAM Identity Center is the recommended way:
aws configure sso --profile dev
aws sso login --profile dev
export AWS_PROFILE=dev          # Windows PowerShell: $env:AWS_PROFILE="dev"

# 3. Install boto3 and check that your credentials work:
pip install boto3
aws sts get-caller-identity     # should print your account ID
```

Tips:
- IDs and names in the examples (bucket names, `123456789012`, `vol-…`) are **placeholders** — replace them with your own.
- Examples that create resources (EC2, SNS, Transcribe…) can cost money; delete what you create.
- Most examples use `eu-north-1` (Stockholm). Change `region_name` to the region you use.

**A first mini-project:** a static website on S3 + CloudFront, then a Lambda + API Gateway + DynamoDB "visitor counter" for it. That touches IAM, S3, Lambda, DynamoDB and CloudWatch — steps 1–6 above.

## 🔀 How a request flows through AWS

```
Browser / Internet
      │
      ▼
  Route 53 (DNS) ....................................................... 01
      │
      ├──► CloudFront ──► S3  (static site / assets) ................... 01, 05
      │
      ├──► WAF + Shield ──► Load Balancer ──► EC2 / ECS / EKS .......... 01, 04
      │                 └► API Gateway   ──► Lambda / Fargate .......... 04
      │
      ▼
  Compute ──► Databases (RDS, Aurora, DynamoDB …) ...................... 06
          ──► Caching (ElastiCache, MemoryDB) .......................... 05
          ──► Storage (S3, EBS, EFS) ................................... 05
          ──► AI (Bedrock, SageMaker …) ................................ 07
          ──► Messaging (SNS, SQS, EventBridge, Step Functions) ........ 08
                    │
                    ▼
  Data lake (S3) ──► Glue / EMR ──► Athena / Redshift ──► Quick Sight .. 09

  Cross-cutting:
    VPC ................................................................ 01
    IAM · Cognito · Secrets Manager .................................... 02
    CloudFormation / CDK · CodePipeline ................................ 03
    CloudWatch · CloudTrail · X-Ray .................................... 10
    Amplify · AppSync · AWS MCP Server ................................. 11
```

Numbers on the right are the file numbers from the **Services by category** table above.

## Entry template

Each service follows the same layout:

- **What it is** — one-line definition
- **Use it for** — typical use cases
- **Key concepts** — the vocabulary you need
- **Pricing model** — what you pay for
- **Gotchas** — common mistakes
- **Python (boto3)** — a minimal example, where useful
