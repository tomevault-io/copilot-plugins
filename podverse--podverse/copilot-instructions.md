## openapi-sync

> Require OpenAPI updates for API surface changes


# OpenAPI Sync Rule

When a change affects HTTP API behavior, update the matching OpenAPI spec in the same change:

- `apps/api/openapi.yml`
- `apps/management-api/openapi.yml`

## Changes that require spec updates

- New/removed/renamed routes or methods
- Request validation changes (required fields, enums, limits, types)
- Response shape or status-code changes
- Authentication or authorization changes (including permission semantics)

## Minimum update requirements

- Add or update `operationId` (unique and stable)
- Keep operation `security` accurate (`security: []` for public endpoints)
- Ensure requestBody and responses are documented
- Add or update at least one realistic example for mutating operations

## Reviewer checks

- Route-to-spec parity is complete
- 401 vs 403 semantics are explicit where relevant
- No stale examples contradict implementation

## Required PR evidence for route-related changes

- `./scripts/nix/with-env npm run openapi:check` output is included in the PR description
- Route parity evidence is included for changed endpoints:
  - changed method+path
  - matching OpenAPI path+method
  - matching `operationId`
- If auth behavior changed, PR includes explicit 401 vs 403 behavior notes
- If a route-related change does not require spec edits, PR includes explicit justification

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
