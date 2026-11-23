# 03 — Infrastructure as Code & CI/CD

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 3.1 | [AWS CloudFormation](#31-aws-cloudformation) | 🟡 Intermediate |
| 3.2 | [AWS CDK (Cloud Development Kit)](#32-aws-cdk-cloud-development-kit) | 🟡 Intermediate |
| 3.3 | [AWS CodeBuild](#33-aws-codebuild) | 🟡 Intermediate |
| 3.4 | [AWS CodeDeploy](#34-aws-codedeploy) | 🔴 Advanced |
| 3.5 | [AWS CodePipeline](#35-aws-codepipeline) | 🟡 Intermediate |

---

## 3.1 AWS CloudFormation

> **Level:** 🟡 Intermediate

**What it is:** AWS's native Infrastructure-as-Code engine — declare resources in YAML/JSON, and CloudFormation creates, updates and deletes them as a unit.

**Key concepts**
- **Template**: `Parameters`, `Mappings`, `Conditions`, `Resources` (required), `Outputs`
- **Stack**: a deployed template; **StackSets** deploy across many accounts/regions
- **Change sets**: preview changes before applying
- **Drift detection**: find manual changes
- Automatic rollback on failure
- Intrinsic functions: `!Ref`, `!GetAtt`, `!Sub`, `!ImportValue`

**Minimal example**
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  AssetsBucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
Outputs:
  BucketName:
    Value: !Ref AssetsBucket
```

**Pricing:** free for AWS resource types (you pay for what it creates).

**Gotchas:** Set `DeletionPolicy: Retain` on databases and buckets you never want deleted by accident.

**Alternatives:** Terraform/OpenTofu (multi-cloud), Pulumi, AWS SAM (CloudFormation extension for serverless).

[↑ Back to contents](#contents)

---

## 3.2 AWS CDK (Cloud Development Kit)

> **Level:** 🟡 Intermediate

**What it is:** Define infrastructure in a real programming language (Python, TypeScript, Java, C#, Go); CDK **synthesizes** it into CloudFormation.

**Key concepts**
- **App → Stacks → Constructs**
- Construct levels: **L1** (raw CloudFormation `Cfn*`), **L2** (sensible defaults, helper methods like `grant_read`), **L3 / patterns** (whole architectures)
- CLI: `cdk init`, `cdk synth`, `cdk diff`, `cdk deploy`, `cdk destroy`
- `cdk bootstrap` once per account/region

**Python example — Lambda behind API Gateway**
```python
from aws_cdk import App, Stack, aws_lambda as _lambda, aws_apigateway as apigw
from constructs import Construct

class ApiStack(Stack):
    def __init__(self, scope: Construct, id: str, **kw):
        super().__init__(scope, id, **kw)
        fn = _lambda.Function(
            self, "Handler",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="app.handler",
            code=_lambda.Code.from_asset("src"),
        )
        apigw.LambdaRestApi(self, "Api", handler=fn)

app = App()
ApiStack(app, "ApiStack")
app.synth()
```

**Pricing:** free.

[↑ Back to contents](#contents)

---

## 3.3 AWS CodeBuild

> **Level:** 🟡 Intermediate

**What it is:** Fully managed build service — runs your build/test commands in a fresh container.

**Key concepts**
- `buildspec.yml` with phases: `install`, `pre_build`, `build`, `post_build`; `artifacts`; `reports`; `cache`
- Managed images or your own Docker image; Linux, Windows, ARM, GPU; can run inside your VPC
- Can act as a **self-hosted GitHub Actions runner**

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - pip install -r requirements.txt
  build:
    commands:
      - pytest --junitxml=reports/junit.xml
      - docker build -t $IMAGE_URI .
      - docker push $IMAGE_URI
reports:
  pytest:
    files: [reports/junit.xml]
```

**Pricing:** per build minute by compute size.

[↑ Back to contents](#contents)

---

## 3.4 AWS CodeDeploy

> **Level:** 🔴 Advanced

**What it is:** Automates application deployments to **EC2/on-prem**, **ECS** and **Lambda**.

**Key concepts**
- `appspec.yml` with lifecycle hooks (`BeforeInstall`, `AfterInstall`, `ApplicationStart`, `ValidateService`…)
- Strategies: **in-place**, **blue/green**, **canary** (e.g. 10% then 90%), **linear**
- Automatic rollback on failure or CloudWatch alarm

**Pricing:** free for EC2/Lambda/ECS; per instance update for on-prem servers.

[↑ Back to contents](#contents)

---

## 3.5 AWS CodePipeline

> **Level:** 🟡 Intermediate

**What it is:** Orchestrates release pipelines (source → build → test → deploy).

**Key concepts**
- **Stages** made of **actions**; actions pass **artifacts** via S3
- Sources: GitHub/GitLab/Bitbucket (via CodeConnections), S3, ECR
- Manual approval actions, triggers on branch/tag, variables

```
Source (GitHub) ─► Build (CodeBuild) ─► Deploy staging (CloudFormation/CodeDeploy)
                                     ─► Manual approval ─► Deploy prod
```

**Pricing:** per active pipeline per month (V1) or per action-execution minute (V2).

**Note:** Many teams use GitHub Actions or GitLab CI with OIDC → IAM role instead; CodePipeline shines when everything stays inside AWS (and with CDK Pipelines).

[↑ Back to contents](#contents)

---

[← 02 Identity, Security & Configuration](02-identity-security-config.md) · [🏠 README](../README.md) · [04 APIs & Compute →](04-apis-and-compute.md)
