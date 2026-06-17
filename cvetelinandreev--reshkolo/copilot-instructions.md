## local-migrations-run

> When working in the local development environment and you add or edit database migration files (or change `schema.prisma` in a way that requires migrations), always run the migration command in the same task instead of only writing SQL.

# Local migration execution rule

When working in the local development environment and you add or edit database migration files (or change `schema.prisma` in a way that requires migrations), always run the migration command in the same task instead of only writing SQL.

Execution policy:
- Preferred: **`npm run db:migrate:dev -- --name <descriptive-name>`** (uses `scripts/with-project-node.sh` so Node matches `.node-version`).
- Non-interactive / CI-style apply of existing migrations: **`npm run db:migrate:deploy`**.
- If those fail, fall back to the same commands with an explicit `bash scripts/with-project-node.sh …` prefix.
- Report clearly which command was used and whether it succeeded.

---
> Source: [cvetelinandreev/reShkolo](https://github.com/cvetelinandreev/reShkolo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
