# Amazon IoT Greengrass (amazon-iot-greengrass)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

AWS IoT Greengrass extends AWS compute, messaging, data management, sync, and ML inference capabilities to edge devices so they can act locally on the data they generate, while still using the cloud for management, analytics, and durable storage.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/amazon-iot-greengrass/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AWS, Edge Computing, IoT, Lambda, Machine Learning, Real-Time Processing

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AWS IoT Greengrass API
The AWS IoT Greengrass V2 API provides access to component management, core device management, and deployment orchestration for edge computing workloads.

**Human URL:** [https://aws.amazon.com/greengrass/](https://aws.amazon.com/greengrass/)

#### Tags:

 - Edge Computing, IoT, Lambda

#### Properties

- [Documentation](https://docs.aws.amazon.com/greengrass/v2/APIReference/)
- [OpenAPI](openapi/amazon-iot-greengrass-openapi-original.yml)
- [GettingStarted](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html)
- [Pricing](https://aws.amazon.com/greengrass/pricing/)
- [FAQ](https://aws.amazon.com/greengrass/faqs/)

## Common Properties

- [Portal](https://aws.amazon.com/greengrass/)
- [Website](https://aws.amazon.com/greengrass/)
- [Documentation](https://docs.aws.amazon.com/greengrass/v2/developerguide/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [Blog](https://aws.amazon.com/blogs/iot/tag/aws-iot-greengrass/)
- [GitHubOrganization](https://github.com/aws)
- [Console](https://console.aws.amazon.com/greengrass/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [Login](https://signin.aws.amazon.com/)
- [StatusPage](https://health.aws.amazon.com/health/status)
- [Contact](https://aws.amazon.com/contact-us/)

## Features

| Name | Description |
|------|-------------|
| Edge Computing | Run Lambda functions and containers on edge devices with local compute. |
| Component System | Deploy reusable software components to edge devices from a component catalog. |
| Local ML Inference | Run machine learning inference locally with SageMaker model deployment. |
| Deployment Management | Deploy and update software components to thousands of edge devices. |
| Local Messaging | Enable MQTT messaging between local IoT devices without cloud round-trips. |

## Use Cases

| Name | Description |
|------|-------------|
| Industrial Edge Processing | Process sensor data locally to reduce latency and bandwidth. |
| Edge ML Inference | Run computer vision and anomaly detection models at the edge. |
| Offline Operation | Continue processing and storing data when disconnected from the cloud. |

## Integrations

| Name | Description |
|------|-------------|
| AWS Lambda | Run Lambda functions on edge devices for local processing. |
| Amazon SageMaker | Deploy trained ML models to edge devices for local inference. |
| AWS IoT Core | Syncs device state and routes messages between edge and cloud. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [AWS IoT Greengrass API](openapi/amazon-iot-greengrass-openapi-original.yml)

### JSON Schema

146 schema files covering key resources and operations.

### JSON Structure

146 JSON Structure files converted from JSON Schema.

### JSON-LD

- [Amazon IoT Greengrass Context](json-ld/amazon-iot-greengrass-context.jsonld)

### Examples

146 example JSON files generated from schemas.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [AWS IoT Greengrass API](capabilities/shared/iot-greengrass.yaml) — operations for amazon iot greengrass management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Edge Device Management](capabilities/edge-device-management.yaml) | Amazon IoT Greengrass | 8 | IoT Developer, Edge Computing Engineer |

## Vocabulary

- [Amazon IoT Greengrass Vocabulary](vocabulary/amazon-iot-greengrass-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas

## Rules

- [Amazon IoT Greengrass Spectral Rules](rules/amazon-iot-greengrass-spectral-rules.yml) — 14 rules across 6 categories enforcing Amazon IoT Greengrass API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
