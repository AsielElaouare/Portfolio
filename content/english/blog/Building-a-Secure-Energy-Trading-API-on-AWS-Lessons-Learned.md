---
title: Building a Secure Energy Trading API on AWS – Lessons Learned
meta_title: "energy trading API"
description: "How we designed, deployed, and operate a real‑time energy trading API on AWS, with OpenAPI, CI/CD, and observability best practices."
image: https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/real-time-order-analytics.png
date: 2026-01-06T19:59:41.913Z
author: "Asiel Elaouare"
tags: ["energy market software", "API operations", "power trading platforms", "AWS API Gateway", "OpenAPI documentation"]
draft: false
---

# Building a Secure Energy Trading API on AWS – Lessons Learned

## Introduction
When I joined the trading platform as a junior developer, my first big responsibility was to help launch a public **energy trading API** that lets market participants pull real‑time order and trade data. The data already lived in our internal analytics warehouse, but exposing it externally required careful attention to security, latency, documentation, and day‑to‑day support. In this post I walk through how we designed, deployed, and operate that API on AWS, and share the practical lessons I picked up along the way.

## Designing the API Contract
### Defining the contract
Our product owners supplied a list of required endpoints – e.g. `GET /v1/orders`, `GET /v1/trades` – plus filter parameters such as `startDate`, `endDate`, and `marketArea`. Because we operate in a regulated market, every field we expose must be traceable to the source system and must respect the regulator’s data‑retention rules. I spent considerable time with the data‑quality team mapping each JSON property to a source column and added a simple `version` field so downstream clients can detect schema changes.

### Choosing OpenAPI 3.0
We selected **OpenAPI 3.0** as the description format because the surrounding tooling (code generators, validation middleware, Redocly) is mature and widely accepted by our external partners. The spec lives in a GitLab repository alongside the Lambda function code, so any contract change is version‑controlled together with the implementation.

## CI/CD Pipeline with GitLab
Our pipeline is intentionally small: `lint → test → package → deploy`.

### Linting with Spectral
The lint stage runs **Spectral** against the OpenAPI file and a custom rule set that enforces the naming conventions required by the compliance team.

### Testing with SAM
The test stage spins up a local SAM (Serverless Application Model) environment, invokes the Lambda handler with synthetic requests, and verifies HTTP status codes and response schemas.

### Packaging and Deployment
Packaging is a simple zip of the Python 3.10 runtime and a few third‑party libraries. Deployment uses GitLab’s built‑in CD runner to invoke an AWS CloudFormation stack that provisions:
- API Gateway
- Lambda function (Python 3.10)
- IAM role with read‑only access to the analytics RDS instance
- VPC endpoint so traffic never leaves the private network
Because the stack is declarative, rolling back is as easy as reverting the commit and letting the pipeline redeploy the previous template.

## Publishing Documentation on S3 & CloudFront
### Automating the docs build
When the OpenAPI file merges to `main`, a second pipeline builds the documentation site with Redocly’s CLI. The CLI reads the spec, injects our branding, and outputs a single HTML file plus a few assets. Those files are copied to an S3 bucket configured for static‑website hosting.

### Handling MIME types and caching
A missing `Content‑Type` header on the S3 object caused browsers to render the HTML as plain text for a few minutes after deployment. Adding a bucket policy that forces the correct MIME type fixed the issue. We also discovered that CloudFront cached an older version of the spec longer than expected; adding a `Cache‑Control: no‑cache` header to the HTML resolved it.

## Using Redocly for Interactive Docs
Redocly does more than generate a pretty page. It validates the spec against the OpenAPI 3.0 schema, warns about unused components, and can generate client SDKs on demand. For our external partners we publish a small JavaScript SDK that wraps `fetch` calls and adds the required HMAC‑signed authentication header. The SDK version is tied to the API version, so a spec bump automatically triggers an npm package bump.

### Practical tip
Keep the spec source in a dedicated folder (e.g. `/api/spec`) and add a pre‑commit hook that runs Spectral. The team receives immediate feedback before a broken contract reaches the pipeline.

## Operational Challenges
Running an API that streams live market data differs from serving a static website. The most common incidents we see are:

### Onboarding delays
New participants need a client certificate, an API key, and a sandbox account. The provisioning workflow was manual, so we built an internal portal that creates the IAM user, stores the certificate in Secrets Manager, and emails a one‑time password. The portal itself is a Lambda function triggered by a DynamoDB stream.

### Data quality spikes
Occasionally the upstream trade feed sends duplicate rows or out‑of‑order timestamps. Because the Lambda queries the analytics view in real time, duplicates appeared in the API response and tripped partner alerts. We added a deduplication layer in the view (`ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY event_time DESC)`) and a health‑check endpoint that returns the last processed event timestamp. Partners can poll this endpoint to verify we are up‑to‑date.

### Rate‑limit abuse
Some participants hammered the endpoint with very small time windows. API Gateway’s built‑in throttling helped, but we also introduced a custom Lambda authorizer that checks each client’s usage pattern stored in DynamoDB and returns a **429** when a daily quota is exceeded.

### Logging and observability
Initially we only logged request IDs to CloudWatch. When an incident occurred we lacked visibility into the query the Lambda executed. Adding structured JSON logs (`requestId`, `clientId`, `query`, `durationMs`) and piping them to an Elasticsearch domain gave us searchable traces and made post‑mortems much faster.

## Key Lessons Learned
1. **Treat the contract as code** – Keeping the OpenAPI spec in the same repo forces discipline; a broken spec is caught early by the CI lint step.
2. **Automate everything** – From documentation builds to client‑certificate provisioning, every manual hand‑off became a source of delay or error.
3. **Don’t ignore CDN caching** – CloudFront improves performance but can hide bugs. Explicit `Cache‑Control` headers and versioned URLs (e.g. `/v1/orders?ts=20240101`) prevent stale data from being served.
4. **Observability is a first‑class feature** – Structured logs, a health‑check endpoint, and CloudWatch dashboards give confidence when answering regulator questions.
5. **Start small, iterate** – The first version exposed only the last 24 hours of orders. Adding a pagination token and a `from` parameter later was straightforward because the underlying view already supported it.

## Closing Thoughts
Working on a public **energy trading API** in a regulated environment forced me to think beyond code. I learned how compliance shapes API design, why CI/CD pipelines must include validation steps that are not “nice to have”, and how a few AWS services can be stitched together to deliver a reliable, documented, and observable service. My next step is to explore automated contract testing with Pact, so downstream partners can verify the API against a shared contract without a full‑stack integration test. If you are starting a similar project, my advice is simple: let the spec drive the implementation, automate the docs, and never underestimate the value of good logging.

