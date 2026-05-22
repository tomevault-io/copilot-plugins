## db

> JoyWork database and Prisma migration rules


# JoyWork Database Rules

- Change data models through `prisma/schema.prisma` plus Prisma migrations; do not rely on manual database edits.
- Preserve the current schema style: singular model names, `camelCase` fields, and explicit `@@map` or `@map` metadata when table naming must stay snake_case.
- For destructive changes like rename, drop, or type narrowing, plan backward compatibility and data migration before removing old paths.
- When schema changes, update affected Prisma queries, selected fields, derived DTOs, and seed logic in the same task.
- Use transactions for multi-step writes that must succeed or fail together, especially when permissions and related records are involved.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
