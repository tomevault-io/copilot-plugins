## database-standards

> Database schema and migration conventions for Backfield


# Database Standards

- Database tables must be namespaced by app prefix such as `agate_` or `stylebook_`.
- Keep schema ownership in `packages/backfield-db`; do not duplicate table contracts across apps.
- Treat indexing as part of schema design. Add indexes intentionally for expected lookup, join, and filter paths.
- Keep model names, migration behavior, and database docs aligned.
- Prefer explicit migrations and readable schema changes over clever shortcuts.

---
> Source: [localangle/backfield](https://github.com/localangle/backfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
