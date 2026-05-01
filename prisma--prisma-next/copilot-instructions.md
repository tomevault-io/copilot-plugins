## cli-e2e-test-patterns

> Patterns for CLI e2e tests using shared fixture app


# CLI E2E Test Fixture Patterns

This rulecard is intentionally short. For full fixture layout and examples, see `docs/reference/cli-e2e-test-patterns.md`.

## Key points

- Prefer the shared fixture app directory for reliable workspace resolution.
- Tests must clean up their own ephemeral directories.

## Related rules

- `.cursor/rules/cli-error-handling.mdc`

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
