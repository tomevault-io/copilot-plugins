## architecture

> JoyWork API module boundaries and shared architecture


# JoyWork API Architecture

- Keep business code inside `src/modules/*`, shared infrastructure in `src/shared`, config in `src/config`, and schema or migrations in `prisma`.
- A feature module should own its `controller`, `service`, `routes`, and `schema` files; avoid cross-module shortcuts that bypass service boundaries.
- Register new modules centrally in `src/app.ts` and keep route prefixes under `/api/...` with Swagger metadata.
- Reuse existing shared helpers like `AppError`, `prisma`, auth, storage, and email services before introducing new infrastructure code.
- Preserve current API contracts for the web app, or update both repos together when a contract must change.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
