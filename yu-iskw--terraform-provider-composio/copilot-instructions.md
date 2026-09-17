## provider-implementation

> Use the HashiCorp Terraform Plugin Framework patterns already present in `internal/provider`.

# Terraform Provider Implementation Guide

Use the HashiCorp Terraform Plugin Framework patterns already present in `internal/provider`.

## Provider

- Keep provider configuration in `internal/provider/provider.go`.
- Use framework `types` in Terraform-facing models.
- Store `*api.Client` in `resp.ResourceData` and `resp.DataSourceData`.
- Keep diagnostics actionable and attribute-specific when possible. Never put API keys in diagnostics.

## Resources

- Put each resource in `internal/provider/resource_<name>.go`.
- Implement `resource.Resource`, `resource.ResourceWithConfigure` when client data is needed, and `resource.ResourceWithImportState` when import is supported.
- Call `internal/composio/api`. Do not build raw HTTP in the provider package.
- Test observable Terraform state behavior.

## Data Sources

- Put each data source in `internal/provider/data_source_<name>.go`.
- Implement `datasource.DataSourceWithConfigure` when client data is needed.
- Prefer deterministic tests using real in-process code and `httptest`.

## Documentation

- Keep examples in `examples/provider`, `examples/resources/<type>`, and `examples/data-sources/<type>`.
- Run `go generate ./...` after schema or example changes.

---
> Source: [yu-iskw/terraform-provider-composio](https://github.com/yu-iskw/terraform-provider-composio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
