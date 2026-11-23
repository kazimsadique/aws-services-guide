# 08 — Application Integration (Coordination)

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 8.1 | [Amazon SNS (Simple Notification Service)](#81-amazon-sns-simple-notification-service) | 🟢 Beginner |
| 8.2 | [Amazon SQS (Simple Queue Service)](#82-amazon-sqs-simple-queue-service) | 🟢 Beginner |
| 8.3 | [Amazon EventBridge](#83-amazon-eventbridge) | 🟡 Intermediate |
| 8.4 | [AWS Step Functions](#84-aws-step-functions) | 🟡 Intermediate |
| 8.5 | [Amazon MWAA (Managed Workflows for Apache Airflow)](#85-amazon-mwaa-managed-workflows-for-apache-airflow) | 🔴 Advanced |

These services **decouple** components so they can scale and fail independently.

| Pattern | Service |
|---------|---------|
| Queue — one consumer group processes each message | **SQS** |
| Pub/sub fan-out — push to many subscribers | **SNS** |
| Event bus with content-based routing, SaaS events, schedules | **EventBridge** |
| Orchestrate multi-step workflows with state, retries, branching | **Step Functions** |
| Scheduled data pipelines (DAGs) written in Python | **MWAA** |

---

## 8.1 Amazon SNS (Simple Notification Service)

> **Level:** 🟢 Beginner

**What it is:** Managed pub/sub messaging — publish once, deliver to many subscribers.

**Key concepts**
- **Topics**: Standard or **FIFO** (ordering + dedup, FIFO subscribers only)
- Subscribers: SQS, Lambda, HTTP/S, email, SMS, mobile push, Kinesis Data Firehose
- **Message filtering** policies per subscription
- **Fan-out pattern**: SNS → multiple SQS queues

**Pricing:** per million publishes + per delivery (varies by protocol; SMS per message).

**Python (boto3) — create a topic, subscribe by email, publish**
```python
import boto3

sns = boto3.client("sns")
topic_arn = sns.create_topic(Name="order-events")["TopicArn"]   # safe to call again

# The recipient must click the confirmation link in the email first
sns.subscribe(TopicArn=topic_arn, Protocol="email", Endpoint="you@example.com")

sns.publish(
    TopicArn=topic_arn,
    Subject="New order",
    Message="Order 1001 was placed.",
    MessageAttributes={"type": {"DataType": "String", "StringValue": "OrderPlaced"}},
)
```

[↑ Back to contents](#contents)

---

## 8.2 Amazon SQS (Simple Queue Service)

> **Level:** 🟢 Beginner

**What it is:** Fully managed message queue.

**Key concepts**
- **Standard** (nearly unlimited throughput, at-least-once, best-effort order) vs **FIFO** (exactly-once processing, ordering per message group)
- **Visibility timeout**: message hidden while a consumer processes it
- **Dead-letter queue (DLQ)** after N failed receives; redrive back to source
- Long polling (`WaitTimeSeconds` up to 20) reduces cost
- Retention up to 14 days, message size up to 1 MiB (larger payloads via S3 pointer pattern)
- Lambda event source mapping with batching and partial batch failure reporting

**Python (boto3)**
```python
import boto3, json

sqs = boto3.client("sqs")
q = sqs.get_queue_url(QueueName="orders")["QueueUrl"]

sqs.send_message(QueueUrl=q, MessageBody=json.dumps({"order_id": 1001}))

msgs = sqs.receive_message(QueueUrl=q, MaxNumberOfMessages=10, WaitTimeSeconds=20).get("Messages", [])
for m in msgs:
    process(json.loads(m["Body"]))  # process() = your own business logic
    sqs.delete_message(QueueUrl=q, ReceiptHandle=m["ReceiptHandle"])  # only after success
```

**Pricing:** per million requests (FIFO slightly more).

**Gotchas:** Consumers must be **idempotent**; set the visibility timeout longer than processing time (≥ 6× Lambda timeout is the usual guidance for Lambda consumers).

[↑ Back to contents](#contents)

---

## 8.3 Amazon EventBridge

> **Level:** 🟡 Intermediate

**What it is:** Serverless event bus for event-driven architectures.

**Key concepts**
- **Event buses**: default (AWS service events), custom, partner (SaaS like Zendesk, Datadog, Shopify)
- **Rules** with **event patterns** (content-based filtering) → **targets** (Lambda, SQS, Step Functions, API destinations, other buses…)
- **EventBridge Scheduler**: cron/rate/one-time schedules at scale (replaces CloudWatch Events cron)
- **Pipes**: point-to-point source → filter → enrich → target (e.g. DynamoDB Stream → Step Functions)
- **Schema registry**, **archive & replay**, cross-account/cross-region delivery

**Example event pattern**
```json
{
  "source": ["com.myapp.orders"],
  "detail-type": ["OrderPlaced"],
  "detail": { "total": [{ "numeric": [">", 1000] }] }
}
```

**Pricing:** per million custom events published (AWS service events to the default bus are free); Scheduler per invocation; Pipes per request.

**SNS vs EventBridge:** SNS for high-throughput simple fan-out and mobile/SMS/email; EventBridge for rich routing, SaaS integration, schemas and replay.

**Python (boto3) — publish a custom event to a bus**
```python
import json
import boto3

events = boto3.client("events")
resp = events.put_events(Entries=[{
    "EventBusName": "orders",
    "Source": "com.myapp.orders",
    "DetailType": "OrderPlaced",
    "Detail": json.dumps({"order_id": 1001, "total": 1499}),
}])
print("Failed entries:", resp["FailedEntryCount"])   # check this — put_events doesn't raise
```

[↑ Back to contents](#contents)

---

## 8.4 AWS Step Functions

> **Level:** 🟡 Intermediate

**What it is:** Visual, serverless workflow orchestration using state machines (Amazon States Language, JSON).

**Key concepts**
- States: Task, Choice, Parallel, **Map** (incl. Distributed Map for millions of items from S3), Wait, Pass, Succeed, Fail
- Built-in **retry/catch** with backoff
- **Standard** workflows (up to 1 year, exactly-once, audit history) vs **Express** (high-volume, up to 5 min, cheaper)
- Direct SDK integrations with 200+ AWS services (no Lambda glue needed)
- Patterns: request-response, **run a job (.sync)**, **wait for callback (task token)** for human approval
- JSONata for data transformation

**Use it for:** order processing, ETL orchestration, ML pipelines, sagas with compensation, human-in-the-loop approvals, orchestrating AI agent steps.

**Pricing:** Standard per state transition; Express per request + duration.

**Python (boto3) — start a workflow and check its result**
```python
import json
import time
import boto3

sfn = boto3.client("stepfunctions")
execution_arn = sfn.start_execution(
    stateMachineArn="arn:aws:states:eu-north-1:123456789012:stateMachine:ProcessOrder",
    input=json.dumps({"order_id": 1001}),
)["executionArn"]

while (run := sfn.describe_execution(executionArn=execution_arn))["status"] == "RUNNING":
    time.sleep(2)
print(run["status"], run.get("output"))
```

[↑ Back to contents](#contents)

---

## 8.5 Amazon MWAA (Managed Workflows for Apache Airflow)

> **Level:** 🔴 Advanced

**What it is:** Managed Apache Airflow — schedule and monitor data pipelines written as Python **DAGs**.

**Key concepts**
- DAGs, plugins and `requirements.txt` stored in S3
- Environment classes (small/medium/large…), auto-scaling workers; **MWAA Serverless** option
- Runs in your VPC; Airflow UI via IAM-authenticated access
- Amazon provider operators for Glue, EMR, Redshift, Athena, SageMaker, etc.

```python
from airflow import DAG
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator
from datetime import datetime

with DAG("daily_sales", start_date=datetime(2026, 1, 1), schedule="@daily", catchup=False):
    GlueJobOperator(task_id="transform_sales", job_name="sales-etl")
```

**Pricing:** per environment-hour (by size) + workers + scheduler + metadata storage. It's always-on, so it's relatively costly for tiny workloads.

**Step Functions vs MWAA:** Step Functions for event-driven, serverless app workflows; MWAA when the team knows Airflow and runs scheduled, data-engineering pipelines with many dependencies.

[↑ Back to contents](#contents)

---

[← 07 AI & Machine Learning](07-ai-ml.md) · [🏠 README](../README.md) · [09 Data Processing & Analytics →](09-data-processing-analytics.md)
