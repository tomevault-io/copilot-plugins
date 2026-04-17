## inspect-action

> - Use modules for coupled components, even if we don't expect to reuse the code. This keeps unrelated pieces of code separated from each other and allows us to use "this" as a resource name more often, for brevity

# Infrastructure as Code

## Terraform Conventions
- Use modules for coupled components, even if we don't expect to reuse the code. This keeps unrelated pieces of code separated from each other and allows us to use "this" as a resource name more often, for brevity
- Keep Lambda functions in `terraform/modules/` (e.g. `terraform/modules/eval_updated`)
- Use `~>` to pin third-party providers and modules to a particular minor version
- Use consistent naming: snake_case for resources, UPPER_CASE for constants

## Lambda Development
- Lambda functions follow the same Python standards as the rest of the codebase

## Docker
- Pin base image versions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/METR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
