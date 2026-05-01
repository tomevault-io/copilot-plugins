## query-patterns

> Standard patterns for working with Prisma Next queries


# Query Patterns

This rulecard is intentionally short. For full examples, see `docs/reference/query-patterns.md`.

## Keep query entrypoints small and explicit

- Prefer a small `db.ts` that wires one `postgres(...)` call and keep query modules consuming `db` directly.
- Keep complex examples and walkthroughs in docs.

## Prefer `tables` + `table.columns.*`

- Access columns via `db.schema.tables.user.columns.email` (avoids conflicts with table properties).
- If a file uses `tables` frequently, extract locals:
  - `const user = db.schema.tables.user`
  - `const cols = user.columns`

## Related rules

- `.cursor/rules/sql-types-imports.mdc`
- `.cursor/rules/validate-contract-usage.mdc`

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
