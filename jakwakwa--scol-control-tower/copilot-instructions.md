## scol-control-tower

> Critical context and instructions for AI agents working in this repository.


Critical context and instructions for AI agents working in this repository.

## Project Overview

**StratCol Onboard Control Tower** is a high-stakes, event-driven onboarding automation platform. It manages the full lifecycle of applicant onboarding—from lead capture to final approval—without middleware.

- **Architecture:** 6-stage pipeline orchestrated by [Inngest](https://www.inngest.com/).
- **Framework:** [Next.js](https://nextjs.org/) (App Router, Turbopack).
- **Runtime & Tooling:** [Bun](https://bun.sh/) is the mandatory package manager and runtime.
- **Database:** [Turso](https://turso.tech/) (LibSQL/SQLite) with [Drizzle ORM](https://orm.drizzle.team/).
- **Auth:** [Clerk](https://clerk.com/).
- **AI Integration:** Multiple Gemini-powered agents (`lib/services/agents/`) for risk, sanctions, and verification.
- **Telemetry:** Extensive PostHog instrumentation with "Perimeter Validation" for security and compliance.

## Building and Running

| Command | Purpose |
|---------|---------|
| `bun dev` | Start Next.js dev server (Turbopack) |
| `bun run dev:all` | Start Next.js + Inngest dev server |
| `bun run build` | Production build |
| `bun run lint` | Run Biome linting |
| `bun run lint:fix` | Biome lint with auto-fix |
| `bun run db:generate` | Generate Drizzle migrations |
| `bun run db:migrate` | Apply migrations to dev DB |
| `bun run db:reset` | Full DB reset (Dev + Test) |
| `bun run test:e2e` | Run Playwright E2E tests |
| `bun run dev:browser-flow` | Start dedicated browser-flow test stack (Port 3100) |

## Development Conventions

### Core Mandates
- **Bun Only:** Use `bun` for all operations. Never use `npm`, `yarn`, or `pnpm`.
- **Strict Typing:** No `any` types. Biome enforces strict correctness.
- **No `console.log`:** Use `console.assert`, `console.warn`, or `console.error`.
- **Server Actions:** Must authenticate internally via `requireAuth()`.
- **Inngest Reliability:** Steps must be replay-safe; deterministic logic only.

### Workflow & Testing
- **Browser-flow Verification:** Required for any changes touching the dashboard or dev server. Use `bun run dev:browser-flow` on port 3100.
- **Database Rules:** Primary data is **SQLite/LibSQL**. Use `json_extract` for JSON fields.
- **Contractual APIs:** Breaking changes to applicant intake (`app/api/applicants`) require explicit approval.
- **Testing Isolation:** `E2E_USE_TEST_DB=1` switches the app to the test database.

### AI Agent Architecture
AI agents are located in `lib/services/agents/` and use Google Gemini:
- **Validation Agent:** Document and data validation.
- **Risk Agent:** Overall risk level assessment (`green`, `amber`, `red`).
- **Sanctions Agent:** Cross-referencing against OpenSanctions and Firecrawl.
- **Reporter Agent:** Generating final compliance reports.

## Key Directories

| Path | Description |
|------|-------------|
| `app/` | Next.js routes; `(authenticated)/` is the dashboard. |
| `inngest/` | Workflow definitions and stage handlers. |
| `lib/services/` | Business logic for risk, sanctions, FICA, etc. |
| `lib/ai/` | GoogleGenAI wrappers and model configurations. |
| `db/schema.ts` | The single-file source of truth for the Drizzle schema. |
| `docs/` | Comprehensive Mintlify documentation. |
| `tests/` | Playwright E2E and staged browser-flow scripts. |

## Documentation (Mintlify)
The documentation is located in `docs/`. To run locally:
1. `bun i -g mint`
2. `cd docs && mint dev`


Learned User Preferences

- Browser-flow and similar UI verification with agent-browser is required before claiming a feature or fix is done—not optional when the work touches the dashboard or dev server behavior.
- Do not run `git commit` or `git push` unless the user explicitly asks.
- For narrow test-fix or debugging tasks, do not change production application code without explicit permission; prefer tests, scripts, env, or docs unless the user expands scope.
- When editing applicant intake (`app/api/applicants`, applicant forms, validations), treat registration and mandate-related API behavior as contractual unless the user approves a breaking change.
- When delegating to long-running subagents (for example deep Inngest or audit passes), wait for them to finish before summarizing, closing the task, or assuming results.
- For `/compound-mintlify` (and the compound-mintlify agent workflow), publish finalized `docs/issues/` markdown to the StratCol AI ClickUp wiki **without** asking “Sync to ClickUp?” when the **`user-clickup`** MCP is available; if MCP is unavailable or the wiki step fails, report that explicitly instead of implying success.

## Learned Workspace Facts

- Cursor agent transcripts for this repo may live under the multi-root workspace project `Users-jakwakwa-Documents-code-workspaces-Scol-Control-Tower-code-workspace` rather than under `Repos/.../Scol-Control-Tower` alone.
- `getDatabaseClient()` uses `TEST_DATABASE_URL` only when `E2E_USE_TEST_DB` equals `1`; Playwright’s web server already sets this for E2E runs.
- Primary app data is **SQLite/LibSQL** (Turso)—**not Postgres**; plans and raw SQL should use SQLite/JSON1 semantics (e.g. `json_extract`), not Postgres-only dialects.
- Browser-flow automation should target the dedicated stack (`bun run dev:browser-flow` / `setup-browser-flow-dev.sh`): load `.env.test` test DB vars, set `E2E_USE_TEST_DB=1`, default `BROWSER_FLOW_PORT` 3100, point Inngest dev’s app URL at the same port as Next, and default `BROWSER_FLOW_SEED_TARGET=test` in `scripts/seed-browser-flow-test-data.ts` so seeded applicants match the test-backed server (override with `app` only when intentionally seeding the dev database).
- Applicant detail URLs support `?tab=` values `overview`, `documents`, `forms`, `risk`, and `reviews` for automation that needs explicit tab state.
- `/dashboard/risk-review/reports/[id]` uses four client-side primary tabs (Procurement, ITC Credit, Sanctions & AML, FICA / KYC) without URL parameters—use full-page screenshots per tab for automation. Manual industry-regulator and social-reputation Firecrawl checks have been **deprecated and removed**; the `POST` routes at `.../checks/industry-regulator` and `.../social-reputation` now return `410 Gone`.
- Shell scripts under `tests/browser-flow/` share `tests/browser-flow/_lib.sh` for env loading, base URL resolution, viewport sizing, `agent-browser screenshot --full`, Clerk login via `input[name="identifier"]` / `input[name="password"]`, and optional `BROWSER_FLOW_UI_APPROVALS=1` for button-driven Stage 5–6 approvals alongside API assertions.
- StratCol AI Wiki issue write-ups from the repo belong under **Control Tower Wiki ⭐️ → Table of Content → Issues [Compound Docs]** (document id `t84m3-19412`); use **`clickup-wiki-stratcol`** after compound-mintlify when MCP is available, per `.cursor/agents/clickup-wiki-stratcol.md` and `.cursor/skills/clickup-mcp/wiki-stratcol-ai.md`.
- Do not hand-edit committed Drizzle migration files; for local or test databases use a clean reset and new migrations through the normal generate path instead of rewriting history. **`bun run db:reset`** drops the dev DB, applies `migrations/` to dev, then—if `.env.test` exists and defines `TEST_DATABASE_URL`—drops the test DB and runs `db:migrate:test` so both databases follow the same SQL migration chain. Use **`bun run test:db:reset`** when you only need to reset the test DB (drop all + `db:migrate:test`). For ad-hoc schema sync without going through migration files, `db:push:test` remains available.
- Legacy SQLite tables **`agents`** and **`xt_callbacks`** were removed from the schema as unused Zapier-era artifacts; current migration baselines do not create them.
- PostHog’s in-app toolbar issues `connect-src` requests to hosts like `https://internal-j.posthog.com`; include **`https://*.posthog.com`** in CSP `connect-src` in `next.config.mjs` alongside `https://us.i.posthog.com` and `https://us-assets.i.posthog.com`.
- `GoogleGenAI` from `@posthog/ai` is typed around `models.generateContent` only; for the Google **Interactions** API (`interactions.create`), use the **`@google/genai`** `GoogleGenAI` client (see `lib/ai/models.ts` for the split).
- **`performAggregatedAnalysis`** in `lib/services/agents/aggregated-analysis.service.ts` is exported but **not** called by Inngest control-tower workflows (stages import individual agents). In production workflow code, the only live Firecrawl surfaces are **VAT** (`runVatVerificationCheck` in Stage 3) and **sanctions fallback** (optional Firecrawl fallback inside `performSanctionsCheck`). Industry-regulator, sanctions-enrichment, and social-reputation Firecrawl helpers have been deprecated and removed.

## Cursor Cloud specific instructions

### Services overview

| Service | Command | Port | Notes |
|---|---|---|---|
| Next.js dev | `bun run dev` | 3000 | Uses Turbopack; see `package.json` scripts for all commands |
| Inngest dev | `bunx inngest-cli@latest dev -u http://localhost:3000/api/inngest` | 8288 | Or use `bun run dev:all` to start both Next.js + Inngest together |
| Biome lint | `bun run lint` | — | Pre-existing lint errors in codebase are normal |

### Gotchas

- **`turbopack.root` in `next.config.mjs`**: This field is hardcoded to the original developer's local path (`/Users/jakwakwa/...`). In Cloud Agent VMs set it to `{}` (empty object) or remove the `root` key so Turbopack infers the project root automatically. Without this fix, both `bun run dev` and `bun run build` will crash with `Invalid distDirRoot` error.
- **Bun is the required runtime**: The lockfile is `bun.lock`; always use `bun install`, `bun run dev`, `bun run build`, etc. Node/npm/pnpm will not resolve dependencies correctly.
- **`.env.local` must exist** with at minimum: `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `DATABASE_URL`, `TURSO_GROUP_AUTH_TOKEN`, `GOOGLE_GENAI_KEY`. Copy from `.env.example` and populate from environment secrets.
- **Database is Turso (remote LibSQL)**: No local DB setup needed; the app connects to the cloud Turso instance via `DATABASE_URL`. Migrations run via `bun run db:migrate`.
- **Clerk auth protects `/dashboard/*` routes**: The landing page at `/` is public; all dashboard routes require Clerk sign-in. Test credentials are needed for E2E tests (`E2E_CLERK_USER_USERNAME`, `E2E_CLERK_USER_PASSWORD`).
- **Lint has pre-existing errors**: `bun run lint` (biome) reports ~55 errors. These are in the existing codebase; do not attempt to fix them unless the user explicitly asks.

---
*Refer to `CLAUDE.md`, and `rules.mdc` for deeper technical rules and learned workspace facts.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jakwakwa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
