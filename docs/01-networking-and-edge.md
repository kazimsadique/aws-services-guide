# 01 — Networking & Edge

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 1.1 | [Amazon Route 53](#11-amazon-route-53) | 🟡 Intermediate |
| 1.2 | [Amazon CloudFront](#12-amazon-cloudfront) | 🟢 Beginner |
| 1.3 | [AWS WAF (Web Application Firewall)](#13-aws-waf-web-application-firewall) | 🟡 Intermediate |
| 1.4 | [AWS Shield](#14-aws-shield) | 🟡 Intermediate |
| 1.5 | [Amazon VPC (Virtual Private Cloud)](#15-amazon-vpc-virtual-private-cloud) | 🟡 Intermediate |
| 1.6 | [AWS VPN](#16-aws-vpn) | 🔴 Advanced |
| 1.7 | [AWS Certificate Manager (ACM)](#17-aws-certificate-manager-acm) | 🟢 Beginner |

These services sit **in front of** your application: they resolve names, cache content close to users, filter bad traffic and define the private network everything runs in.

---

## 1.1 Amazon Route 53

> **Level:** 🟡 Intermediate

**What it is:** A managed DNS service with domain registration and health checks.

**Use it for**
- Registering domains and hosting DNS zones
- Routing users to CloudFront, load balancers, API Gateway or S3
- Failover between regions and blue/green cutovers

**Key concepts**
- **Hosted zone**: public (internet) or private (inside a VPC)
- **Record types**: A, AAAA, CNAME, MX, TXT, plus AWS **Alias** records (point at AWS resources, free queries, work at the zone apex)
- **Routing policies**: simple, weighted, latency-based, failover, geolocation, geoproximity, multivalue, IP-based
- **Health checks**: probe endpoints and drive failover routing

**Pricing:** per hosted zone per month + per million queries (Alias queries to AWS resources are free) + health checks.

**Gotchas**
- You can't put a CNAME at the zone apex (`example.com`) — use an Alias record.
- Low TTLs speed up failover but increase query cost.

[↑ Back to contents](#contents)

---

## 1.2 Amazon CloudFront

> **Level:** 🟢 Beginner

**What it is:** AWS's global content delivery network (CDN).

**Use it for**
- Serving static websites and assets from S3
- Caching API responses at edge locations
- Video streaming and large file downloads
- TLS termination close to users

**Key concepts**
- **Distribution**: the CDN config; has one or more **origins** (S3, ALB, API Gateway, any HTTP server)
- **Cache behaviors**: path patterns (`/api/*`, `/static/*`) with their own cache and origin settings
- **Origin Access Control (OAC)**: lets CloudFront read a private S3 bucket so users can't bypass the CDN
- **CloudFront Functions** (lightweight JS at the edge) and **Lambda@Edge** (heavier logic)
- **Invalidations**: purge cached objects

**Pricing:** data transfer out + requests; invalidations beyond a free quota. Flat-rate plans are also available.

**Gotchas**
- Certificates for CloudFront must be in ACM **us-east-1**.
- Forgetting cache keys (headers/cookies/query strings) leads to either poor hit rates or users seeing each other's data.

**Python (boto3) — clear the CDN cache after deploying a new site version**
```python
import time
import boto3

cf = boto3.client("cloudfront")
cf.create_invalidation(
    DistributionId="E1234567890ABC",                 # your distribution ID
    InvalidationBatch={
        "Paths": {"Quantity": 1, "Items": ["/*"]},   # everything; or e.g. "/index.html"
        "CallerReference": str(time.time()),         # must be unique per request
    },
)
```

[↑ Back to contents](#contents)

---

## 1.3 AWS WAF (Web Application Firewall)

> **Level:** 🟡 Intermediate

**What it is:** A layer-7 firewall that inspects HTTP(S) requests.

**Use it for**
- Blocking SQL injection, XSS and other OWASP Top 10 attacks
- Rate limiting by IP
- Geo-blocking, bot control, IP allow/deny lists

**Key concepts**
- **Web ACL** attached to CloudFront, ALB, API Gateway, AppSync or Cognito
- **Rules / rule groups**, including **AWS Managed Rules**
- Actions: Allow, Block, Count, CAPTCHA, Challenge

**Pricing:** per Web ACL + per rule + per million requests inspected; extra for Bot Control and similar managed groups.

**Gotchas:** Start new rules in **Count** mode to avoid blocking real users.

[↑ Back to contents](#contents)

---

## 1.4 AWS Shield

> **Level:** 🟡 Intermediate

**What it is:** DDoS protection.

| Tier | What you get | Cost |
|------|--------------|------|
| **Shield Standard** | Automatic layer 3/4 protection for all AWS customers | Free |
| **Shield Advanced** | Enhanced detection, layer-7 DDoS mitigation with WAF, 24/7 Shield Response Team, cost protection for scaling during attacks | Monthly subscription + data transfer fees |

**Use Advanced for:** business-critical, publicly exposed workloads where downtime is very expensive.

[↑ Back to contents](#contents)

---

## 1.5 Amazon VPC (Virtual Private Cloud)

> **Level:** 🟡 Intermediate

**What it is:** Your own isolated network inside AWS.

**Key concepts**
- **CIDR block** (e.g. `10.0.0.0/16`)
- **Subnets**: public (route to an Internet Gateway) vs private (no direct inbound internet), each in one Availability Zone
- **Internet Gateway (IGW)**: internet access for public subnets
- **NAT Gateway**: outbound-only internet for private subnets
- **Route tables**
- **Security Groups**: stateful, instance-level firewall (allow rules only)
- **Network ACLs**: stateless, subnet-level firewall (allow + deny)
- **VPC Endpoints**: private access to AWS services (Gateway endpoints for S3/DynamoDB; Interface endpoints / PrivateLink for others)
- **VPC Peering** and **Transit Gateway**: connect VPCs together
- **Flow Logs**: log network traffic metadata

**Typical layout**
```
VPC 10.0.0.0/16
├── Public subnets  (ALB, NAT Gateway)          — AZ a, AZ b
├── Private subnets (EC2 / ECS / Lambda)        — AZ a, AZ b
└── Isolated subnets (RDS, ElastiCache)         — AZ a, AZ b
```

**Pricing:** the VPC itself is free; you pay for NAT Gateways, Interface Endpoints, Transit Gateway, public IPv4 addresses and data transfer.

**Gotchas**
- NAT Gateway data processing charges add up — use S3/DynamoDB Gateway Endpoints (free) for that traffic.
- Plan CIDR ranges so they don't overlap with on-prem or other VPCs you may peer with later.

[↑ Back to contents](#contents)

---

## 1.6 AWS VPN

> **Level:** 🔴 Advanced

**What it is:** Encrypted connectivity into your VPC.

| Option | Purpose |
|--------|---------|
| **Site-to-Site VPN** | IPsec tunnels from an office/data center router to a Virtual Private Gateway or Transit Gateway |
| **Client VPN** | Managed OpenVPN-based access for individual users' laptops |

**Alternative:** **AWS Direct Connect** — a dedicated private line for consistent bandwidth and latency.

**Pricing:** per connection-hour + data transfer (Client VPN: per endpoint association + per active connection).

[↑ Back to contents](#contents)

---

## 1.7 AWS Certificate Manager (ACM)

> **Level:** 🟢 Beginner

**What it is:** Free public TLS/SSL certificates with automatic renewal, for use with AWS services.

**Use it for:** HTTPS on CloudFront, ALB/NLB, API Gateway, App Runner, etc.

**Key concepts**
- **DNS validation** (recommended — auto-renews if the CNAME stays in place) or email validation
- **Private CA** (AWS Private CA) for internal certificates — paid
- Public certs are regional; CloudFront needs them in **us-east-1**

**Gotchas:** Standard ACM certificates can't be exported to install on your own EC2 web server (exportable public certificates are a separate paid option).

**Python (boto3) — request a free certificate for CloudFront**
```python
import time
import boto3

acm = boto3.client("acm", region_name="us-east-1")   # CloudFront requires us-east-1
arn = acm.request_certificate(
    DomainName="example.com",
    SubjectAlternativeNames=["www.example.com"],
    ValidationMethod="DNS",
)["CertificateArn"]

time.sleep(10)  # the DNS record details take a few seconds to appear
cert = acm.describe_certificate(CertificateArn=arn)["Certificate"]
record = cert["DomainValidationOptions"][0]["ResourceRecord"]
print("Add this CNAME in Route 53:", record["Name"], "->", record["Value"])
```

[↑ Back to contents](#contents)

---

[🏠 README](../README.md) · [02 Identity, Security & Configuration →](02-identity-security-config.md)
