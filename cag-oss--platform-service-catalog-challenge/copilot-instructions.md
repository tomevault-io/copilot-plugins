## platform-service-catalog-challenge

> Build a small internal developer platform service that lets development teams register services and publish operational health updates through a self-service API.

# AI Agent Instructions

## Project Goal

Build a small internal developer platform service that lets development teams register services and publish operational health updates through a self-service API.

## Technical Constraints

- Use TypeScript on AWS Lambda behind API Gateway HTTP API.
- Persist catalog data in DynamoDB.
- Manage AWS resources with Terraform modules.
- Keep IAM least privilege and avoid wildcard resource policies.
- Do not hardcode secrets or sensitive values.
- Use structured JSON logging for operational visibility.
- Add tests for validation and behavior changes.

## Agent Workflow Rules

- Prefer small, reviewable changes.
- Explain tradeoffs in pull request summaries.
- Course-correct generated Terraform that introduces broad IAM permissions.
- Keep the service intentionally small unless a requirement justifies more scope.
- Update README or DECISIONS.md when behavior, deployment, or architecture changes.

## Human Review Checklist

- Are API errors normalized and useful?
- Are write endpoints validated?
- Are IAM permissions scoped to named resources?
- Can the service be demonstrated with simple curl commands?
- Is there a clear explanation of AI-assisted development decisions?

---
> Source: [cag-oss/platform-service-catalog-challenge](https://github.com/cag-oss/platform-service-catalog-challenge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
