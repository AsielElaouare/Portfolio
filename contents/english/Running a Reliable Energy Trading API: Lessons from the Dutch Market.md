---
title: Running a Reliable Energy Trading API: Lessons from the Dutch Market
meta_title: "energy trading API"
description: "How we keep a regulated Dutch energy trading API highly available with CI/CD, AWS, OpenAPI and smart automation."
date: 2026-01-06T14:07:10.835Z
author: "Asiel Elaouare"
draft: false
---
# Running a Reliable Energy Trading API: Lessons from the Dutch Market

*Outline*
- Why availability matters more than performance
- CI/CD pipeline: small steps, fast feedback
- What worked & what still needs work
- AWS hosting: the invisible glue (incidents & fixes)
- Documentation: the contract we all rely on
- Day‑to‑day operational chores
- Lessons learned
- Looking ahead

When I first joined the **API Operations** team, I was thrilled to work on a product that literally keeps the lights on for an entire country. The public **energy trading API** we expose is the only way external market participants—brokers, risk managers, and data vendors—can retrieve real‑time order and trade information from the Dutch energy market. It sounds glamorous, but the day‑to‑day reality is a series of tiny fixes, careful monitoring, and constant learning about how regulated, high‑availability services are built.

## Why Availability Trumps Performance

In a typical web startup you hear a lot about latency and throughput. For us, the most important metric is *availability*: a missing trade record can break a downstream risk calculation, cause a compliance breach, or even trigger financial penalties. The market regulator expects us to publish every order and trade within a few seconds of it happening, and they audit us regularly. That pressure shapes every technical decision—from the way I write a CI pipeline to how I version the OpenAPI spec.

## CI/CD Pipeline: Small Steps, Fast Feedback

We run everything on **GitLab CI/CD** runners. The original `.gitlab-ci.yml` file listed dozens of jobs—lint, unit test, contract test, security scan, build Docker image, push to ECR, deploy to ECS, invalidate CloudFront, and finally run a post‑deployment health check. Over the past six months I have simplified the flow without sacrificing safety.

### What Worked

- **Separate linting from testing** – Running `spectral` on the OpenAPI definition in its own job catches documentation errors early. I added a rule that fails the job if any `x‑regulatory` extensions are missing, which helped us enforce a small but important compliance rule.
- **Docker multi‑stage builds** – By compiling the Go service in a builder stage and copying only the binary into a tiny Alpine image, we reduced the container size from 350 MB to 45 MB. Smaller images mean faster pulls and a smaller attack surface.
- **Canary deployments** – Instead of a full blue‑green switch, we now deploy to a “canary” ECS task set that receives 5 % of the traffic via a weighted Route 53 record. If the health check passes for 10 minutes we promote the canary to 100 %. This gave us confidence to roll out schema changes without taking the whole API down.

### What I Still Wrestle With

The biggest pain point is the long feedback loop when a change touches both the backend and the documentation site. The docs are built with **Redocly** and hosted on S3 + CloudFront. After a successful backend deploy I have to run a separate job that runs `redocly build-docs`, uploads the static HTML to the bucket, and finally invalidates the CloudFront distribution. The invalidate call can take up to 15 minutes, and during that window some users still see the old spec. I’m experimenting with a versioned bucket (e.g., `api-docs/v2/`) and a small Lambda@Edge that redirects to the latest version based on a header. It adds complexity, but it could eliminate the race condition entirely.

## AWS Hosting: The Invisible Glue

Our API runs in a private subnet behind an **Application Load Balancer (ALB)**. The ALB terminates TLS, forwards traffic to ECS tasks, and provides health‑check endpoints that our GitLab monitor pings every minute. A few incidents taught me why the “invisible” parts of the architecture are just as important as the code.

### Incident #1 – S3 Bucket Versioning Misconfiguration

A client reported that a newly added field `priceUnit` was missing from the JSON payload. The backend was correctly sending the field, but the response schema on the documentation site still described the old version. The S3 bucket used for the docs had versioning disabled, so a previous upload had overwritten the file. The fix was to enable versioning, add a lifecycle rule to retain the last 10 versions, and update the deployment script to always upload with a unique object key (`api-docs-{{CI_COMMIT_SHA}}.html`). This incident reminded me that data‑durability rules apply to static assets just as they do to trade logs.

### Incident #2 – CloudFront Cache‑Stale Error Responses

During a scheduled maintenance window we rotated the TLS certificate on the ALB. The API returned HTTP 502 for a few minutes, but CloudFront kept serving the cached 502 response for up to 30 seconds after the ALB was back online. Adding a custom header (`Cache‑Control: no‑store`) to error responses prevented CloudFront from caching them, stopping a cascade of alerts from downstream systems that kept retrying the API.

## Documentation: The Contract We All Rely On

In a regulated market the **OpenAPI definition** is more than a developer convenience; it is a legal contract. Redocly gives us a nice UI, but the real work happens in the spec itself.

- **Schema versioning** – We follow a strict “major.minor” scheme. A breaking change (e.g., removing a field) bumps the major version and forces all clients to update. Minor changes (adding optional fields) can be rolled out with a backward‑compatible release. The CI pipeline enforces that a PR cannot merge if the `info.version` field does not follow the expected pattern.
- **Regulatory extensions** – The regulator requires us to tag every endpoint with `x‑regulatory: true` and to provide a `x‑data‑retention` field indicating how long the data is stored. I wrote a custom Spectral rule that fails the build if any endpoint is missing these extensions.
- **Testing the contract** – Using `prism` we spin up a mock server from the OpenAPI file and run integration tests against it. This catches mismatches between the spec and the actual implementation before the code even reaches a staging environment.

## Day‑to‑Day Operational Chores

Beyond the big incidents, most of my time is spent on repetitive but essential tasks:

- **Onboarding new market participants** – They receive a sandbox API key, a copy of the latest spec, and a checklist that includes IP whitelisting, rate‑limit configuration, and a test order flow. Automating the checklist with a small Lambda reduced onboarding time from two days to a few hours.
- **Monitoring data quality** – We run a nightly Spark job that compares the number of trades recorded in our internal database with the number of records the API served. Any discrepancy triggers a PagerDuty alert. The last time this caught a bug was when a serialization library dropped fields with a `null` value.
- **Edge‑case debugging** – Occasionally a client sends a request with an unusually large `pageSize` (10 000). The backend respects the limit, but the response payload grows to 12 MB, exceeding the ALB’s max payload size. I added a defensive guard that caps `pageSize` at 2 000 and returns a clear error message. The change was small, but it prevented a cascade of failed requests from a handful of clients.

## Lessons Learned (and What I Still Need to Improve)

1. **Treat the spec as code** – Version control, linting, and automated tests are non‑negotiable. The spec is the single source of truth for both developers and regulators.
2. **Fail fast, recover fast** – Canary deployments and health‑check alerts give us early warning before a problem reaches a client.
3. **Cache is a double‑edged sword** – CloudFront improves latency, but you must think about stale error responses and versioned assets.
4. **Automation pays dividends** – Even a simple Lambda that creates a sandbox user saves hours of manual work each month.
5. **Never underestimate edge cases** – The market is full of legacy systems that send odd requests. Defensive validation and clear error messages keep the API usable for everyone.

## Looking Ahead

My next goal is to add a lightweight observability layer that correlates API request latency with downstream market‑data spikes. If we can see that a sudden surge in order volume is causing latency spikes, we could automatically scale the ECS service before the SLA is breached. It’s a classic “predict‑and‑prevent” problem, and I’m excited to apply what I’ve learned about metrics, alarms, and automated remediation.

Working on a public **energy trading API** in a regulated market has taught me that reliability is a habit, not a feature. Every commit, every doc change, every alert is an opportunity to keep the lights on—for our customers and for the market as a whole.

