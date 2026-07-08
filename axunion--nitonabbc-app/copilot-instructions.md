## nitonabbc-app

> This file provides guidance to Claude Code when working with this repository.

# AGENTS.md

This file provides guidance to Claude Code when working with this repository.

> **Keep in sync**: `CLAUDE.md` and `AGENTS.md` must always have identical content. When editing this file, apply the same changes to `CLAUDE.md`.

## Project Overview

PWA for members (~30) of Nitonabbc Church. Solid.js + TypeScript, Vite build, pnpm.

- LINE authentication only, 2 roles (admin / member)
- Light theme + frost-white glassmorphism ("God's Glory" theme), mobile-first
- Japanese/English i18n
- External subdomain services embedded via iframe

## Approach

Bias toward caution over speed; on trivial tasks, use judgment.

- **Think before coding.** State assumptions; if uncertain or multiple interpretations exist, surface them rather than silently picking one. Push back when a simpler path exists.
- **Simplest thing that works.** Write the minimum code that solves the stated problem — no speculative abstractions, flexibility, or error handling for impossible cases.
- **Surgical changes.** Every changed line should trace to the request. Don't refactor or reformat adjacent code that isn't broken; match the surrounding style. Remove only the symbols your change orphans; leave unrelated dead code and mention it.
- **Goal-driven.** Turn each task into a verifiable outcome (e.g. "fix the bug" → write a failing test, then make it pass). For multi-step work, state a brief plan with a verification check per step.

## Language Policy

All files in this repository — including `CLAUDE.md`, `.claude/` configs, source code comments, commit messages, and dev console output — are written in **English**.

## Specification Documents

Specs live in `docs/`. Always read the relevant doc before implementing.
Add a new doc in `docs/` and link it from `spec.md` when adding features.

@docs/spec.md

## Safety: Deployment and Production Data

Production deployments and remote DB migrations run exclusively via GitHub Actions.
The npm scripts `deploy` and `db:migrate:remote` are **intentionally not defined** in
`package.json` to prevent accidental local execution.

**Never run the following from a local machine:**

- `wrangler deploy` — production deployments are triggered by push to `main` (or any branch for Preview)
- `wrangler d1 migrations apply ... --remote` — production DB migrations run via GitHub Actions only
- Any other `wrangler` command targeting production with `--remote`

**Allowed locally:**

- `pnpm db:migrate:local` — apply pending migrations to local D1
- `pnpm db:reset` — wipe local D1 and re-apply all migrations from scratch
- `pnpm db:seed` — insert sample data from `scripts/seed.sql`
- `pnpm db:fresh` — `db:reset` + `db:seed` in one step
- All other local dev operations: `pnpm dev`, `pnpm build`, `pnpm check`, `pnpm test`, etc.

## Architecture

- **Entry**: `index.html` → `src/index.tsx` → `src/App.tsx`
- **Routing**: `@solidjs/router`. Routes defined in `src/index.tsx`, root layout in `src/App.tsx` (auth gate + Toaster)
  - Member routes (`/`, `/settings`, `/bulletin/*`) nest under `MemberLayout` (TabBar)
  - Admin routes (`/admin/*`) nest under `AdminLayout` (sidebar, admin role guard)
  - TabBar: bottom pill on mobile (<900px), vertical sidebar on desktop (≥900px). Each tab remembers and restores its last URL (iOS UITabBarController equivalent)
- **Path alias**: `@/` → `./src` (configured in both `vite.config.ts` and `tsconfig.app.json`)
- **Auth context**: `useAuth()` from `src/store/AuthContext.tsx` — provides user info and `logout`
- **JSX**: Solid.js transform (`jsxImportSource: solid-js`)
- **Deploy**: Cloudflare Workers + static assets

See `.claude/rules/` for detailed conventions per area (UI, CSS, API, testing).

## Directory Structure

- `src/components/<Name>/` — component, CSS module, and `index.ts` re-export
- `src/pages/<Name>/` — page component (route-level screen), with `hooks/` for complex logic
- `src/api/<resource>.ts` — API fetch functions (pages do not write fetch logic inline)
- `src/store/` — Solid.js stores (`createResource`-based)
- `src/styles/` — global styles (`tokens.css`, `reset.css`, `shared.module.css`)
- `server/routes/` — Hono API routes (one file per resource)
- `server/middleware/` — shared middleware
- `server/db/schema.ts` — Drizzle schema (single source of truth)
- `server/db/types.ts` — shared bulletin section types
- `server/db/index.ts` — `createDb()` and `Db` type
- `server/types.ts` — `AppEnv`, `User`, `SessionData`, and other server-wide types
- `worker/index.ts` — Worker entry point (re-exports the Hono app)
- `drizzle/` — drizzle-kit generated migration SQL (committed)

## Environments

| Env | Description |
|-----|-------------|
| Local | `pnpm dev` starts Vite (HMR) + workerd together via `@cloudflare/vite-plugin`. Set `DEV_AUTH=true` in `.dev.vars` to skip LINE auth and auto-login as admin |
| Preview | Auto-deployed on non-`main` branch push |
| Production | Auto-deployed on `main` branch push |

KV / D1 bindings are isolated per environment. LINE Login callback URLs must be configured per environment.

## Commands

- `pnpm check` — Biome lint/format + TypeScript check
- `pnpm test` — run tests (Vitest, `server/**/*.test.ts` only)

## Database (Cloudflare D1 + Drizzle)

- Schema: `server/db/schema.ts` is the single source of truth
- Access DB via `c.get("db")` — never use `c.env.DB` directly in routes or middleware
- Migrations: `pnpm db:generate` → commit `drizzle/` → `pnpm db:migrate:local`
- Local DB commands:
  - `pnpm db:reset` — wipe local D1 + re-apply all migrations
  - `pnpm db:seed` — insert sample data (`scripts/seed.sql`)
  - `pnpm db:fresh` — `db:reset` + `db:seed` in one step
- Use `/db-migrate` skill for guided schema change workflow
- Production migrations run via GitHub Actions only (never `--remote` locally)

## Code Structure

- Name variables, functions, and files to communicate intent.
- Extract a helper only when used in 3+ places; otherwise inline it.
- One concern per file; split when a file exceeds ~300 lines.
- Delete dead code; never comment it out.
- Biome config in `biome.json` (indentStyle:space, quoteStyle:double, VCS integration, organizeImports)

## Testing

- **TDD**: write the test first (Red → Green → Refactor). See `.claude/rules/testing.md`
- Test observable outcomes and edge cases, not implementation details.
- Each test must be fully self-contained; no shared mutable state between tests.

## Commits

Format:

```
<one-line summary>

<Why: one sentence — motivation or problem>

- <change 1>
- <change 2>
```

- Summary: imperative mood, ≤70 chars, no trailing period, no prefix tags (`feat:`, `fix:`, etc.).
- Why line: include only when motivation is not evident from the diff alone.
- Bullets: include only for 2+ distinct changes.
- Never commit secrets (`*.key`, `*.pem`, `credentials*`).
- Never use `--no-verify` or `--amend`; always create a new commit.

## Claude Code Automation

Configs in `.claude/rules/`, agents in `.claude/agents/`, skills in `.claude/skills/`.

### Agents

| Agent | When to use |
|-------|-------------|
| `security-reviewer` | Changing auth, sessions, invite links, or admin routes |

### Skills

| Skill | Purpose |
|-------|---------|
| `/spec-update` | Sync `docs/spec.md` and individual feature docs |
| `/db-migrate` | Guided workflow for D1 schema changes |
| `/new-feature` | Scaffold a brand-new feature across all layers (schema → API → UI → i18n → docs) |

---
> Source: [axunion/nitonabbc-app](https://github.com/axunion/nitonabbc-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
