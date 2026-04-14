## it-tms

> High-level architecture and monorepo conventions

# Architecture

- **Monorepo** managed by **Turborepo** with **pnpm workspaces**.
- Apps live in `apps/`:
  - `apps/web` — Next.js (App Router) + TypeScript + Tailwind v4 + HeroUI.
  - `apps/api` — Go + Fiber + PostgreSQL.
- Shared packages in `packages/`:
  - `packages/config` — ESLint Flat, Prettier, base TS config.
  - `packages/ui` — Theme tokens + UI helpers.
- Database migrations are stored at repo root under `db/migrations` (consumed by the API).

**Versions**
- Node: LTS (>= 20), pnpm 9.x
- Go: latest stable (>= 1.22)
- Postgres: 16+
- Tailwind: v4
- HeroUI: latest (`@heroui/react`)

**Commands**
- `pnpm dev` runs both apps in parallel via `turbo`.
- `pnpm build|lint|test|format|typecheck` delegates to each workspace.

**Security by default**
- CORS allowlist from env.
- Helmet secure headers.
- Rate-limits on Auth & Ticket creation.
- JWT with role claims; httpOnly cookie on the web origin.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/starmoneypb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/starmoneypb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
