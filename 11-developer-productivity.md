# 11 — Developer Productivity

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 11.1 | [AWS Amplify](#111-aws-amplify) | 🟢 Beginner |
| 11.2 | [AWS AppSync](#112-aws-appsync) | 🟡 Intermediate |
| 11.3 | [Amazon WorkSpaces](#113-amazon-workspaces) | 🟢 Beginner |
| 11.4 | [Amazon Q → Kiro (AI assistants)](#114-amazon-q--kiro-ai-assistants) | 🟢 Beginner |
| 11.5 | [AWS MCP Server (Model Context Protocol)](#115-aws-mcp-server-model-context-protocol) | 🟡 Intermediate |

---

## 11.1 AWS Amplify

> **Level:** 🟢 Beginner

**What it is:** A toolkit and hosting service for building full-stack web and mobile apps quickly.

**Key concepts**
- **Amplify Hosting**: Git-based CI/CD and hosting for static sites and SSR frameworks (Next.js, Nuxt, Astro…), preview branches, custom domains
- **Amplify Gen 2**: define the backend in **TypeScript** (code-first, built on CDK) — auth (Cognito), data (AppSync + DynamoDB), storage (S3), functions (Lambda)
- Client libraries for React, Vue, Angular, Swift, Android, Flutter, React Native
- UI components (e.g. Authenticator)

**Use it for:** frontend-heavy apps, MVPs, quick SaaS prototypes.

**Pricing:** hosting build minutes + storage + data served; backend resources billed at their normal rates.

**Trade-off:** Very fast to start; for complex backends many teams move to plain CDK.

[↑ Back to contents](#contents)

---

## 11.2 AWS AppSync

> **Level:** 🟡 Intermediate

**What it is:** Managed **GraphQL** API service, plus serverless **pub/sub (Events)** WebSocket APIs.

**Key concepts**
- **Schema** (types, queries, mutations, subscriptions)
- **Resolvers** written in JavaScript (APPSYNC_JS) or VTL; pipeline resolvers
- **Data sources**: DynamoDB, Lambda, RDS/Aurora (Data API), OpenSearch, HTTP, EventBridge, Bedrock
- **Real-time subscriptions** over WebSockets
- **AppSync Events**: simple channel-based pub/sub without GraphQL
- Auth: API key, IAM, Cognito, OIDC, Lambda authorizer
- Caching, **Merged APIs** for federating multiple teams' schemas

**Pricing:** per million queries/mutations + real-time messages and connection-minutes.

**API Gateway vs AppSync:** REST/HTTP APIs → API Gateway; GraphQL, real-time sync, mobile/offline clients → AppSync.

[↑ Back to contents](#contents)

---

## 11.3 Amazon WorkSpaces

> **Level:** 🟢 Beginner

**What it is:** Managed virtual desktops (DaaS) — Windows, Linux or Ubuntu desktops streamed to any device.

**Variants**
- **WorkSpaces Personal**: persistent desktop per user
- **WorkSpaces Pools**: non-persistent, shared pools
- **WorkSpaces Secure Browser** (formerly WorkSpaces Web): managed browser for accessing internal web apps
- **WorkSpaces Thin Client**: low-cost hardware device

**Use it for:** contractors, remote developers needing a secure dev environment, call centers, BYOD, data that must not leave the cloud.

**Pricing:** monthly or hourly per desktop bundle (CPU/RAM/storage), plus licensing.

[↑ Back to contents](#contents)

---

## 11.4 Amazon Q → Kiro (AI assistants)

> **Level:** 🟢 Beginner

**What it is:** AWS's generative-AI assistant brand.

**Current status (important):**
- **Amazon Q Developer IDE plugins** and paid subscriptions reach **end of support on 30 April 2027**; new sign-ups stopped on 15 May 2026. AWS's successor is **Kiro**, a spec-driven agentic IDE and CLI (built on Code OSS, so VS Code settings/extensions carry over).
- The former **Amazon Q CLI** is now the **Kiro CLI**.
- **Amazon Q Developer in the AWS Console**, docs site, Console Mobile App, and in chat apps (Slack, Microsoft Teams) is not affected by the end-of-support notice.
- **Amazon Q Business** (enterprise knowledge assistant) has largely been folded into **AWS Quick** (see file 09).

**Kiro key ideas:** specs (requirements → design → tasks), agent hooks, steering files, MCP support.

**Use it for:** asking about AWS resources and costs in the console, troubleshooting errors, generating/refactoring code, upgrading Java/.NET (AWS Transform).

[↑ Back to contents](#contents)

---

## 11.5 AWS MCP Server (Model Context Protocol)

> **Level:** 🟡 Intermediate

**What it is:** A **managed, remote MCP server** that gives AI agents and coding assistants (Claude Code, Kiro, Cursor, Codex, etc.) secure, authenticated access to AWS. It became generally available in **May 2026** (preview at re:Invent 2025) and is part of the **Agent Toolkit for AWS** (MCP server + skills + plugins).

**Key concepts**
- A small, fixed set of tools — e.g. a `call_aws` tool that can execute any of the 15,000+ AWS API operations with your IAM credentials — plus searching up-to-date AWS documentation
- Auth: **IAM + SigV4**, via the `mcp-proxy-for-aws` local proxy; IAM condition keys can restrict what agents may do; actions are auditable in CloudTrail
- Regions (at GA): US East (N. Virginia) and Europe (Frankfurt) endpoints, e.g. `https://aws-mcp.eu-central-1.api.aws/mcp`
- It replaces the open-source `aws-api-mcp-server` and `aws-knowledge-mcp-server` — remove those from your MCP config to avoid tool-name conflicts
- The **awslabs/mcp** GitHub repo still hosts many specialised open-source servers (DynamoDB data modeling, Serverless/SAM, Redshift, AppSync, IaC, etc.)

**Example (Claude Code)**
```bash
claude mcp add-json aws-mcp --scope user \
  '{"command":"uvx","args":["mcp-proxy-for-aws@latest","https://aws-mcp.eu-central-1.api.aws/mcp","--metadata","AWS_REGION=eu-north-1"]}'
```

**Gotchas:** Give the agent a **least-privilege IAM role** (read-only to start); never point it at production credentials with write access without guardrails.

[↑ Back to contents](#contents)

---

[← 10 Monitoring & Logging](10-monitoring-and-logging.md) · [🏠 README](../README.md) · [12 Reference Architectures & Cheat Sheet →](12-reference-architectures.md)
