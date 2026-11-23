# 10 — Monitoring & Logging

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 10.1 | [Amazon CloudWatch](#101-amazon-cloudwatch) | 🟢 Beginner |
| 10.2 | [AWS CloudTrail](#102-aws-cloudtrail) | 🟢 Beginner |
| 10.3 | [AWS Config](#103-aws-config) | 🟡 Intermediate |
| 10.4 | [AWS X-Ray](#104-aws-x-ray) | 🟡 Intermediate |

| Question | Service |
|----------|---------|
| Is my app healthy? Metrics, logs, alarms, dashboards | **CloudWatch** |
| **Who** did **what** API call, **when**? | **CloudTrail** |
| **What** does my resource configuration look like, and is it compliant? | **AWS Config** |
| **Where** is a request slow across services? | **X-Ray** (traces) |

---

## 10.1 Amazon CloudWatch

> **Level:** 🟢 Beginner

**What it is:** The central observability service — metrics, logs, alarms, dashboards, and more.

**Key concepts**
- **Metrics**: AWS service metrics automatically; custom metrics; **Embedded Metric Format (EMF)** to emit metrics through logs
- **Alarms** → SNS, Auto Scaling, EC2 actions; composite alarms; anomaly detection
- **Logs**: log groups/streams, retention settings, **Logs Insights** query language, metric filters, subscription filters (to Lambda/Firehose/OpenSearch), live tail
- **Dashboards**
- **CloudWatch Agent**: OS metrics and log files from EC2/on-prem
- **Application Signals** (APM/SLOs), **Synthetics** canaries, **RUM** (real-user monitoring), **Container Insights**, **Lambda Insights**, **Transaction Search** for traces

**Logs Insights example**
```
fields @timestamp, @message
| filter level = "ERROR"
| stats count() by bin(5m)
```

**Pricing:** custom metrics, alarms, dashboards, API calls, log ingestion (per GB) + storage + Logs Insights scanned data.

**Gotchas:** Log groups default to **never expire** — set retention. Verbose debug logging can cost more than the compute.

**Python (boto3) — publish a custom metric and search your logs**
```python
import time
import boto3

cw = boto3.client("cloudwatch")
cw.put_metric_data(
    Namespace="MyApp",
    MetricData=[{
        "MetricName": "OrdersPlaced",
        "Value": 1,
        "Unit": "Count",
        "Dimensions": [{"Name": "Environment", "Value": "prod"}],
    }],
)

# Logs Insights: count errors in the last hour
logs = boto3.client("logs")
now = int(time.time())
qid = logs.start_query(
    logGroupName="/aws/lambda/hello",
    startTime=now - 3600,
    endTime=now,
    queryString='fields @timestamp, @message | filter @message like /ERROR/ | stats count()',
)["queryId"]
time.sleep(3)
print(logs.get_query_results(queryId=qid)["results"])
```

[↑ Back to contents](#contents)

---

## 10.2 AWS CloudTrail

> **Level:** 🟢 Beginner

**What it is:** Records AWS API activity (console, CLI, SDK, services) for auditing, security and troubleshooting.

**Key concepts**
- **Event history**: last 90 days of management events, free, per region
- **Trails**: deliver events to S3 (and optionally CloudWatch Logs); create an **organization trail** for all accounts
- Event types: **management events** (control plane), **data events** (e.g. S3 object reads, Lambda invokes — extra cost), **Insights events** (unusual activity)
- **CloudTrail Lake**: managed, SQL-queryable event store
- Log file integrity validation

**Use it for:** "who deleted that security group?", compliance audits, incident response, GuardDuty input.

**Pricing:** first copy of management events to S3 is free; data events, extra copies, Insights and Lake are charged.

**Python (boto3) — find out who deleted something**
```python
import boto3

ct = boto3.client("cloudtrail")
events = ct.lookup_events(
    LookupAttributes=[{"AttributeKey": "EventName", "AttributeValue": "DeleteBucket"}],
    MaxResults=10,
)["Events"]
for e in events:
    print(e["EventTime"], e.get("Username"), e["EventName"])
```

[↑ Back to contents](#contents)

---

## 10.3 AWS Config

> **Level:** 🟡 Intermediate

**What it is:** Continuously records resource configurations and evaluates them against rules.

**Key concepts**
- **Configuration items** and timeline history per resource
- **Config rules**: AWS managed (e.g. `s3-bucket-public-read-prohibited`, `encrypted-volumes`) or custom (Lambda / Guard)
- **Conformance packs**: bundles of rules (CIS, PCI…)
- **Remediation** via Systems Manager Automation
- **Aggregators** for multi-account/multi-region views
- Advanced queries (SQL over current configuration)

**Pricing:** per configuration item recorded + per rule evaluation + conformance pack evaluations.

**Related:** **Security Hub** aggregates Config findings with GuardDuty, Inspector, etc.

[↑ Back to contents](#contents)

---

## 10.4 AWS X-Ray

> **Level:** 🟡 Intermediate

**What it is:** Distributed tracing — follow a request through API Gateway, Lambda, containers, databases and downstream calls.

**Key concepts**
- **Traces** made of **segments/subsegments** (OpenTelemetry: spans)
- **Service map / trace map** showing latency and errors between components
- **Sampling rules** to control volume and cost
- Native integrations: Lambda, API Gateway, Step Functions, App Runner, ECS/EKS via collectors
- Now viewed largely through the CloudWatch console (Application Signals, Transaction Search)

**Important change:** The X-Ray **SDKs and daemon** entered maintenance mode on 25 Feb 2026 and reach end-of-support on 25 Feb 2027. The X-Ray *service* itself remains supported; AWS recommends instrumenting new code with **OpenTelemetry** — e.g. **AWS Distro for OpenTelemetry (ADOT)** — and sending traces to X-Ray.

**Python:** use the ADOT Python auto-instrumentation (`opentelemetry-instrument`) or, in Lambda, Powertools Tracer / the ADOT Lambda layer.

**Pricing:** per million traces recorded and retrieved/scanned (free tier available).

[↑ Back to contents](#contents)

---

[← 09 Data Processing & Analytics](09-data-processing-analytics.md) · [🏠 README](../README.md) · [11 Developer Productivity →](11-developer-productivity.md)
