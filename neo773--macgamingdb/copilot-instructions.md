## macgamingdb

> Monorepo: Next.js web app (`src/`) + NestJS tRPC API (`packages/macgamingdb-server`, port 4000) + UI primitives (`packages/macgamingdb-ui`) + emails (`packages/macgamingdb-emails`). Bun workspaces.

# MacGamingDB

Monorepo: Next.js web app (`src/`) + NestJS tRPC API (`packages/macgamingdb-server`, port 4000) + UI primitives (`packages/macgamingdb-ui`) + emails (`packages/macgamingdb-emails`). Bun workspaces.

**Read `docs/CODE_STYLE.md` before writing any code. Its rules are hard lines, not suggestions.**

- Dev: `bun run dev:server` (API on :4000) + `bun run dev` (Next; falls back past :3000 if occupied — never kill what holds :3000).
- Verify: `bunx tsc --noEmit` at root AND in `packages/macgamingdb-server`; `bun run test`; `bun run lint`.
- DB: drizzle; local db at `packages/macgamingdb-server/prisma/dev.db`; migrations via `bun run db:generate` / `db:migrate`. Never run anything against `.env.prod` unprompted.
- OpenAPI for the iOS app: `bun --bun scripts/generate-openapi.ts` after router/dto changes.
- Do not commit `MobileAppSwift/` or `TWENTY_*_CONVENTIONS.md`.

---
> Source: [neo773/macgamingdb](https://github.com/neo773/macgamingdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
