# 02 — Identity, Security & Configuration

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 2.1 | [AWS IAM (Identity and Access Management)](#21-aws-iam-identity-and-access-management) | 🟢 Beginner |
| 2.2 | [AWS IAM Identity Center (successor to AWS SSO)](#22-aws-iam-identity-center-successor-to-aws-sso) | 🟢 Beginner |
| 2.3 | [Amazon Cognito](#23-amazon-cognito) | 🟡 Intermediate |
| 2.4 | [AWS Secrets Manager](#24-aws-secrets-manager) | 🟡 Intermediate |
| 2.5 | [AWS AppConfig](#25-aws-appconfig) | 🔴 Advanced |

Rule of thumb:
- **IAM / Identity Center** → who in *your organization* (people and services) can touch AWS
- **Cognito** → the *end users of your app* signing in
- **Secrets Manager / AppConfig** → what your app needs to know at runtime

---

## 2.1 AWS IAM (Identity and Access Management)

> **Level:** 🟢 Beginner

**What it is:** The permission system for every AWS API call.

**Key concepts**
- **Users** (long-term credentials — avoid for humans now; use Identity Center)
- **Groups**: collections of users
- **Roles**: identities assumed temporarily by services, apps, or federated users — the main building block
- **Policies** (JSON): `Effect`, `Action`, `Resource`, `Condition`
  - Identity-based vs resource-based (e.g. S3 bucket policy)
  - Permission boundaries, Service Control Policies (SCPs, from AWS Organizations)
- **STS**: issues temporary credentials when a role is assumed
- Evaluation: explicit **Deny** wins → otherwise needs an explicit **Allow** → default deny

**Example policy (read one S3 bucket)**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
  }]
}
```

**Pricing:** free.

**Best practices**
- Least privilege; use IAM Access Analyzer to find unused permissions
- Never use the root account day-to-day; enable MFA
- Give workloads roles (EC2 instance profile, ECS task role, Lambda execution role), never access keys

**Python (boto3) — Who am I? (check which identity your code is using)**
```python
import boto3

me = boto3.client("sts").get_caller_identity()
print(me["Account"], me["Arn"])   # e.g. arn:aws:sts::123456789012:assumed-role/Dev/kazi

# List the roles in the account
iam = boto3.client("iam")
for page in iam.get_paginator("list_roles").paginate():
    for role in page["Roles"]:
        print(role["RoleName"])
```

[↑ Back to contents](#contents)

---

## 2.2 AWS IAM Identity Center (successor to AWS SSO)

> **Level:** 🟢 Beginner

**What it is:** Central sign-in for people across multiple AWS accounts and business apps.

**Use it for**
- One login for all accounts in an AWS Organization
- Connecting an external identity provider (Entra ID / Azure AD, Okta, Google Workspace)
- Short-lived CLI credentials: `aws configure sso` → `aws sso login`

**Key concepts:** permission sets (templates turned into roles in each account), account assignments, SCIM user provisioning.

**Pricing:** free.

**Python (boto3) — use your Identity Center (SSO) login from Python**
```python
# One-time setup in a terminal:
#   aws configure sso --profile dev      (answer the prompts)
#   aws sso login --profile dev          (opens the browser)
import boto3

session = boto3.Session(profile_name="dev")   # short-lived credentials, no access keys
s3 = session.client("s3")
print([b["Name"] for b in s3.list_buckets()["Buckets"]])
```

[↑ Back to contents](#contents)

---

## 2.3 Amazon Cognito

> **Level:** 🟡 Intermediate

**What it is:** Authentication and user management for your web and mobile apps.

**Key concepts**
- **User Pools**: user directory + sign-up/sign-in, MFA, password policies, hosted/managed login UI, social and SAML/OIDC federation. Issues **JWTs** (ID, access, refresh tokens).
- **Identity Pools**: exchange a token for temporary **AWS credentials** so the client can call AWS directly (e.g. upload to S3)
- Lambda triggers (pre-sign-up, post-confirmation, custom auth…)
- Integrates natively with API Gateway and AppSync authorizers

**Pricing:** per monthly active user (MAU), with feature tiers (Lite, Essentials, Plus).

**Gotchas**
- User pool attributes are hard to change after creation — plan the schema.
- Exporting users with passwords isn't possible; migration requires a migration trigger.

**Python: validate a Cognito JWT in a FastAPI backend** — use the JWKS at
`https://cognito-idp.<region>.amazonaws.com/<user_pool_id>/.well-known/jwks.json` with a library like `python-jose` or `PyJWT`.

[↑ Back to contents](#contents)

---

## 2.4 AWS Secrets Manager

> **Level:** 🟡 Intermediate

**What it is:** Encrypted storage for secrets (DB passwords, API keys) with automatic rotation.

**Use it for**
- Database credentials with automatic rotation (native for RDS/Aurora/Redshift/DocumentDB)
- Third-party API keys
- Cross-account secret sharing

**Pricing:** per secret per month + per 10,000 API calls.

**Alternative:** **SSM Parameter Store** — cheaper/free for plain config and simple secrets, but no built-in rotation.

**Python (boto3)**
```python
import boto3, json

sm = boto3.client("secretsmanager")
secret = json.loads(sm.get_secret_value(SecretId="prod/db")["SecretString"])
db_password = secret["password"]
```
Cache secrets in memory (e.g. the `aws-secretsmanager-caching` library or Lambda extension) rather than calling on every request.

[↑ Back to contents](#contents)

---

## 2.5 AWS AppConfig

> **Level:** 🔴 Advanced

**What it is:** Part of AWS Systems Manager — safely deploy application configuration and **feature flags** without redeploying code.

**Use it for**
- Feature flags and gradual rollouts
- Operational toggles (kill switches, throttling limits)
- Allow-lists, tuning parameters

**Key concepts**
- Application → Environment → Configuration profile
- **Validators** (JSON Schema or Lambda) reject bad configs
- **Deployment strategies** (e.g. linear 10% every minute) with automatic **rollback on CloudWatch alarm**
- Retrieve via the AppConfig Agent (Lambda extension / sidecar) or the `appconfigdata` API

**Pricing:** per configuration request and per configuration received.

[↑ Back to contents](#contents)

---

[← 01 Networking & Edge](01-networking-and-edge.md) · [🏠 README](../README.md) · [03 Infrastructure as Code & CI/CD →](03-iac-and-cicd.md)
