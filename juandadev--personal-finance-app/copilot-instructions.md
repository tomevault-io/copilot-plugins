## protect-dev-database

> The database in `.env.local` contains real user data, not disposable fixtures.

# Protect Dev Database

The database in `.env.local` contains real user data, not disposable fixtures.

- Never run `bun run db:seed`, data-mutating scripts, or manual SQL writes
  against it without explicit permission in the current chat.
- `bun run db:migrate` may be run when a task requires a new migration, but
  migrations must be replay-safe (the runner re-executes every file) and must
  never rewrite or reassign rows owned by existing users.
- Never create test accounts or insert test rows in this database without
  asking first. Prefer asking the user to verify flows themselves.
- Before any data repair, run it as a dry run inside a transaction and show
  the output before applying.

---
> Source: [juandadev/personal-finance-app](https://github.com/juandadev/personal-finance-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
