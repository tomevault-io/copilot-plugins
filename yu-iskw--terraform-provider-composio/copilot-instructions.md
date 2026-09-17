## project-structure

> - `internal/provider`: Terraform provider implementation, resources, data sources, embedded docs, and tests.

# Project Structure Guide

- `internal/provider`: Terraform provider implementation, resources, data sources, embedded docs, and tests.
- `internal/composio/api`: Composio REST client for `/api/v3.1`.
- `internal/composio/models`: Domain models without Terraform types.
- `examples`: Terraform examples used by documentation generation.
- `docs`: Provider documentation.
- `tools`: Go tool dependency tracking.
- `main.go`: Provider server entry point.
- `go.mod` and `go.sum`: Go module and dependency metadata.
- `.github`: CI and release workflows.
- `.trunk`: Tooling configuration.

---
> Source: [yu-iskw/terraform-provider-composio](https://github.com/yu-iskw/terraform-provider-composio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
