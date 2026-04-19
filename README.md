# Amazon IoT Greengrass (amazon-iot-greengrass)
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
