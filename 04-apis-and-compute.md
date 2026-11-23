# 04 — APIs & Compute

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| **Part A** | **[Front doors (APIs)](#part-a--front-doors-apis)** | |
| 4.1 | [Elastic Load Balancing (ELB)](#41-elastic-load-balancing-elb) | 🟡 Intermediate |
| 4.2 | [Amazon API Gateway](#42-amazon-api-gateway) | 🟢 Beginner |
| **Part B** | **[Compute](#part-b--compute)** | |
| — | [Choosing compute](#choosing-compute) | Overview |
| 4.3 | [Amazon EC2 (Elastic Compute Cloud)](#43-amazon-ec2-elastic-compute-cloud) | 🟢 Beginner |
| 4.4 | [Amazon Lightsail](#44-amazon-lightsail) | 🟢 Beginner |
| 4.5 | [Amazon ECS (Elastic Container Service)](#45-amazon-ecs-elastic-container-service) | 🟡 Intermediate |
| 4.6 | [AWS Fargate](#46-aws-fargate) | 🟡 Intermediate |
| 4.7 | [Amazon EKS (Elastic Kubernetes Service)](#47-amazon-eks-elastic-kubernetes-service) | 🔴 Advanced |
| 4.8 | [AWS Lambda](#48-aws-lambda) | 🟢 Beginner |

---

## Part A — Front doors (APIs)

### 4.1 Elastic Load Balancing (ELB)

> **Level:** 🟡 Intermediate

**What it is:** Distributes incoming traffic across targets in multiple Availability Zones.

| Type | Layer | Best for |
|------|-------|----------|
| **Application Load Balancer (ALB)** | 7 (HTTP/HTTPS, gRPC, WebSockets) | Web apps, microservices; path/host-based routing |
| **Network Load Balancer (NLB)** | 4 (TCP/UDP/TLS) | Very high throughput, static IPs, non-HTTP protocols |
| **Gateway Load Balancer (GWLB)** | 3 | Inserting third-party firewalls/appliances |

**Key concepts:** listeners, rules, **target groups** (EC2, IPs, Lambda, ECS tasks), health checks, sticky sessions, cross-zone balancing.

**Pricing:** per hour + Load Balancer Capacity Units (LCUs).

[↑ Back to contents](#contents)

---

### 4.2 Amazon API Gateway

> **Level:** 🟢 Beginner

**What it is:** Managed front door for APIs — routing, auth, throttling, and more.

| Flavor | Notes |
|--------|-------|
| **HTTP API** | Cheaper, faster, simpler; JWT authorizers; most new REST-style APIs |
| **REST API** | Feature-rich: API keys + usage plans, request validation, transformations, caching, WAF, private APIs |
| **WebSocket API** | Real-time bidirectional apps (chat, live dashboards) |

**Key concepts:** stages, integrations (Lambda, HTTP, AWS services, VPC Link to private ALB/NLB), authorizers (IAM, Cognito, Lambda, JWT), throttling.

**Pricing:** per million requests (+ data transfer; WebSocket per message and connection-minute).

**Gotchas:** 29-second default integration timeout (can be raised for REST APIs in some cases) — long work should go async (SQS/Step Functions).

**ALB vs API Gateway:** ALB for container/EC2 fleets with steady traffic; API Gateway for serverless, per-request pricing, and API-management features.

**Python (boto3) — put an HTTP API in front of an existing Lambda function**
```python
import boto3

region, account = "eu-north-1", "123456789012"
fn_arn = f"arn:aws:lambda:{region}:{account}:function:hello"

apigw = boto3.client("apigatewayv2", region_name=region)
api = apigw.create_api(Name="hello-api", ProtocolType="HTTP", Target=fn_arn)  # "quick create"

# Allow API Gateway to invoke the function
boto3.client("lambda", region_name=region).add_permission(
    FunctionName="hello",
    StatementId="allow-api-gateway",
    Action="lambda:InvokeFunction",
    Principal="apigateway.amazonaws.com",
    SourceArn=f"arn:aws:execute-api:{region}:{account}:{api['ApiId']}/*",
)
print("Call it at:", api["ApiEndpoint"])
```

[↑ Back to contents](#contents)

---

## Part B — Compute

### Choosing compute

```
Full control of OS, GPUs, long-running processes?    → EC2                         4.3
Just a small VPS-like server, predictable price?     → Lightsail                   4.4
Containers, AWS-native and simple?                   → ECS (on Fargate or EC2)     4.5
Containers without managing servers?                 → Fargate (with ECS or EKS)   4.6
Containers, need Kubernetes ecosystem/portability?   → EKS                         4.7
Event-driven, short tasks, scale-to-zero?            → Lambda                      4.8
```

### 4.3 Amazon EC2 (Elastic Compute Cloud)

> **Level:** 🟢 Beginner

**What it is:** Virtual machines in the cloud.

**Key concepts**
- **Instance types**: families such as `t` (burstable), `m` (general), `c` (compute), `r` (memory), `g/p` (GPU), `i` (storage). Graviton (ARM, `g` suffix like `m7g`) is usually cheaper per performance.
- **AMI**: machine image
- **EBS volumes** for disks, **instance store** for ephemeral disk
- **Security groups**, **key pairs** / **SSM Session Manager** (no SSH port needed)
- **User data**: bootstrap script at launch
- **Auto Scaling Groups** + **Launch Templates**
- **Placement groups**, Elastic IPs

**Pricing models**
| Model | Discount | When |
|-------|----------|------|
| On-Demand | — | Spiky / short-term |
| Savings Plans / Reserved Instances | up to ~72% | Steady baseline, 1–3 yr commit |
| Spot | up to ~90% | Fault-tolerant batch jobs (can be interrupted with 2-min notice) |

**Gotchas:** Patch the OS yourself; use multiple AZs; stop paying for forgotten instances and unattached EBS volumes.

**Python (boto3) — launch, wait for, and terminate a small server**
```python
import boto3

region = "eu-north-1"
ec2 = boto3.client("ec2", region_name=region)
ssm = boto3.client("ssm", region_name=region)

# Always get the latest Amazon Linux 2023 image ID instead of hard-coding it
ami = ssm.get_parameter(
    Name="/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64"
)["Parameter"]["Value"]

resp = ec2.run_instances(
    ImageId=ami,
    InstanceType="t3.micro",
    MinCount=1,
    MaxCount=1,
    TagSpecifications=[{"ResourceType": "instance",
                        "Tags": [{"Key": "Name", "Value": "my-first-server"}]}],
)
instance_id = resp["Instances"][0]["InstanceId"]
ec2.get_waiter("instance_running").wait(InstanceIds=[instance_id])
print("Running:", instance_id)

ec2.terminate_instances(InstanceIds=[instance_id])  # clean up so you don't keep paying
```

[↑ Back to contents](#contents)

---

### 4.4 Amazon Lightsail

> **Level:** 🟢 Beginner

**What it is:** Simplified VPS hosting with bundled, predictable monthly pricing (instance + storage + transfer).

**Use it for:** WordPress, small websites, dev/test servers, simple apps; also offers managed databases, containers, load balancers and CDN in simplified form.

**Trade-off:** Less flexibility than EC2; can peer with a VPC or export snapshots to EC2 when you outgrow it.

**Python (boto3) — list your Lightsail servers and their public IPs**
```python
import boto3

ls = boto3.client("lightsail", region_name="eu-north-1")
for inst in ls.get_instances()["instances"]:
    print(inst["name"], inst["state"]["name"], inst.get("publicIpAddress"))
```

[↑ Back to contents](#contents)

---

### 4.5 Amazon ECS (Elastic Container Service)

> **Level:** 🟡 Intermediate

**What it is:** AWS's own container orchestrator — simpler than Kubernetes.

**Key concepts**
- **Cluster**, **Task definition** (container image, CPU/memory, env vars, ports, IAM **task role**), **Task**, **Service** (keeps N tasks running, integrates with ALB, auto scaling)
- Capacity: **Fargate** (serverless) or **EC2** (you manage instances; capacity providers)
- Images usually from **Amazon ECR**
- Service Connect for service-to-service discovery

**Pricing:** no extra charge for ECS itself; pay for Fargate or EC2.

[↑ Back to contents](#contents)

---

### 4.6 AWS Fargate

> **Level:** 🟡 Intermediate

**What it is:** Serverless compute engine for containers — run tasks/pods without managing servers.

**Use with:** ECS (most common) or EKS.

**Pricing:** per vCPU-second and GB-second; **Fargate Spot** for up to ~70% off interruptible tasks.

**Trade-offs:** no host access, no GPUs; slightly more expensive per unit than well-packed EC2, but no ops overhead.

[↑ Back to contents](#contents)

---

### 4.7 Amazon EKS (Elastic Kubernetes Service)

> **Level:** 🔴 Advanced

**What it is:** Managed Kubernetes control plane.

**Key concepts**
- Data plane options: **managed node groups**, self-managed nodes, **Fargate profiles**, **Karpenter** autoscaling, **EKS Auto Mode** (AWS manages nodes, networking, storage add-ons)
- **IAM ↔ Kubernetes**: EKS Pod Identity or IRSA for per-pod AWS permissions
- Add-ons: VPC CNI, CoreDNS, kube-proxy, EBS CSI driver, AWS Load Balancer Controller

**Pricing:** per cluster-hour for the control plane + nodes/Fargate.

**When:** team already knows Kubernetes, needs Helm/operators ecosystem, or multi-cloud portability. Otherwise ECS is usually simpler.

[↑ Back to contents](#contents)

---

### 4.8 AWS Lambda

> **Level:** 🟢 Beginner

**What it is:** Run functions in response to events without servers; scales to zero.

**Key concepts**
- **Triggers**: API Gateway, ALB, S3, SQS, SNS, EventBridge, DynamoDB/Kinesis streams, schedules, function URLs
- Limits: up to **15 minutes** per invocation, up to 10 GB memory (CPU scales with memory), `/tmp` up to 10 GB
- Packaging: zip or container image (up to 10 GB); **Layers** for shared dependencies
- **Concurrency**: reserved / provisioned concurrency; **SnapStart** for faster cold starts (Java, Python, .NET)
- Execution role (IAM), environment variables, VPC attachment, DLQs / on-failure destinations

**Python handler**
```python
import json

def handler(event, context):
    body = json.loads(event.get("body") or "{}")
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"hello": body.get("name", "world")}),
    }
```
Tip: **Powertools for AWS Lambda (Python)** gives logging, tracing, metrics, validation and idempotency helpers.

**Pricing:** per request + GB-second of duration (generous free tier).

**Gotchas:** cold starts; make handlers **idempotent** (events can be delivered more than once); initialize SDK clients outside the handler; watch for recursive loops (e.g. S3 trigger writing to the same bucket).

[↑ Back to contents](#contents)

---

[← 03 Infrastructure as Code & CI/CD](03-iac-and-cicd.md) · [🏠 README](../README.md) · [05 Storage & Caching →](05-storage-and-caching.md)
