## wacha

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Wacha ("Worker Aggregation and Control Hub for Agents") is an MCP server (Streamable HTTP transport) plus a React SPA web UI served by the same Hono server, for orchestrating AI agents through Project / Story / Task workflows with role-based access and review gates. See `ARCHITECTURE.md`, `DOMAIN.md`, `AGENTS.md`, and `REVIEW_POLICY.md` for the full design rationale — this file only covers what's needed to work in the code.

Note: `knowledge/` and `skill/` are MCP-served content for agents (via `list_skills`/`get_skill_context`), NOT development guidelines for this repo itself.

## Commands

```bash
npm install
npm run start         # prestart builds the SPA into public/, then runs src/server.ts via tsx (:51743)
npm run build         # frontend typecheck (tsc -p frontend) + vite build into public/
npm run dev           # tsx watch (:51743) + Vite dev server (:5173, proxies /api) in parallel
npm test              # resets .tmp/wacha-test.db, runs all tests via node --test --test-concurrency=1
```

Run a single test file directly (test DB env var is required, or tests will hit `wacha.db`):

```bash
WACHA_DB_PATH=.tmp/wacha-test.db node --import tsx --test test/mcp/tool/IssueStoryTool.test.ts
```

There is no separate lint/typecheck script; use `npx tsc --noEmit` if you need to check types.

Env vars: `PORT` (default 51743), `WACHA_DB_PATH` (default `wacha.db`). `.env` is auto-loaded if present (`src/bootstrap/loadEnv.ts`).

## Architecture

Layering and dependency direction (enforced by convention, not tooling):

```
presentation / mcp  →  application  →  domain  ←  infrastructure
```

- `src/domain` — models, repository *interfaces*, domain services. Must never import MCP, Hono, SQLite, or filesystem code.
- `src/application/usecase` — one class per use case (e.g. `IssueStoryUseCase`, `ClaimTaskUseCase`). This is the unit of change; avoid grab-bag services like a generic `TaskService`.
- `src/application/service` — cross-cutting orchestration (`SessionService`, `MembershipService`, `InstructionService`), not use-case-specific.
- `src/infrastructure/repository` — Kysely/better-sqlite3 implementations of the domain repository interfaces; `src/infrastructure/database` holds the schema and `initializeSchema()`.
- `src/mcp/tool` — one file per MCP tool, each exporting `{ config, execute }`. Tools are thin adapters: validate input (zod), call exactly one use case, return via `toMcpErrorResponse`/`toTextResult`. No business logic here.
- `src/mcp/middleware/RoleGuard.ts` — `withRoleGuard(allowRoles, context, execute)` wraps a tool's `execute` to enforce `ProjectRole` checks against the caller's `sessionId`.
- `src/mcp/createMcpServer.ts` — the single place tools are registered on the `McpServer` and wired to `withRoleGuard`. When adding a tool, register it here and add it to the role table in `agent/role-policy.md` and `AGENTS.md`.
- `src/container.ts` — manual DI: repositories are instantiated once, use cases are constructed from them and exported as singletons. New use cases/repositories are wired here, not with a framework.
- `src/presentation/controller/PageController.ts` — JSON API controller for the web UI (`/api/*` routes), calling the same use cases as the MCP tools. Response/request shapes are typed against `shared/apiTypes.ts`.
- `src/app.ts` — `createApp()` builds the Hono app (static serving, `/api` routes, `/mcp`, `/health`, SPA fallback, path-branched `onError`: REST shape `{error:{message}}` for `/api`, JSON-RPC shape for everything else). `src/server.ts` only bootstraps and calls `serve()`. Tests exercise routes via `createApp().request(...)`.
- `frontend/` — React 19 + Vite + TanStack Query + Tailwind v4 SPA. Single npm package: all deps live in the root package.json; `frontend/` only holds source, `vite.config.ts` (with `root` set to its own dir), and its own tsconfig. `vite build` emits into `public/`, which Hono serves with an `index.html` fallback for unknown GET paths. Dev: Vite on :5173 proxies `/api` to Hono. Data flow: one aggregate query per project (`useProject`, 5s polling — agents mutate state via MCP at any time) and mutations that invalidate it (`frontend/src/lib/queries.ts`).
- `shared/apiTypes.ts` — self-contained DTO/wire types shared by server (`@shared/*` alias) and frontend. Keep it free of imports from `src/` or `frontend/`.

Path aliases (see `tsconfig.json`): `@container`, `@application/*`, `@constants/*`, `@mcp/*`, `@domain/*`, `@repository/*` (→ `infrastructure/repository`), `@database/*` (→ `infrastructure/database`), `@bootstrap/*`, `@controller/*`, `@shared/*` (→ `../shared`). Imports use explicit `.ts` extensions (NodeNext module resolution). The frontend has its own tsconfig (`moduleResolution: bundler`, aliases `@/*` and `@shared/*`).

### Identity model

There are no user accounts. Agent identity and project-role membership are keyed by MCP `sessionId`. Sessions are lost on server restart; clients must re-`initialize` and re-`assign_project_role` — there is no session/role recovery logic to build.

### Task/Story lifecycle

```
todo -> doing -> in_review -> wait_accept -> accepted
                    ↓
                 rejected
```

Roles: `manager` (planning + final accept/reject), `reviewer` (technical validation, moves `in_review` → `wait_accept` or `reject`), `worker` (execution, `claim_task`/`complete_task`). Story `doing` is driven by a worker claiming its first task, not set directly. The authoritative role/tool permission table is in `agent/role-policy.md` — update it (and `AGENTS.md`'s tool list) whenever tool access changes, since these docs are served to agents at runtime via `get_role_instructions`.

Cancellation (`cancel_story`/`cancel_task`) is meant to be a non-destructive status transition with a reason left as a comment, not a hard delete — the existing delete use cases are legacy and tracked separately from this policy.

### Skills vs. tools

`skill/*.md` files are reusable, descriptive operating procedures for agents (served via `list_skills`/`get_skill_context`, backed by `FileSkillRepository`), distinct from MCP tools which are executable. Don't blend the two: a skill only becomes a tool once it's stable and needs to be directly invokable.

## Testing conventions

- Tests use Node's built-in `node:test` + `node:assert/strict`, one `test/` tree mirroring `src/`.
- Tests that touch the DB call `initializeSchema()` and clear relevant tables in `beforeEach` (see `test/mcp/tool/IssueStoryTool.test.ts`) — order matters due to foreign keys (`task` → `project_membership`/`story` → `project`).
- `WACHA_DB_PATH` must point at a scratch DB (`.tmp/wacha-test.db`); `npm test` handles this for you, but running a file directly requires setting it yourself.
- `--test-concurrency=1` is intentional (shared SQLite file) — don't parallelize test runs.

---
> Source: [akihiroxob/wacha](https://github.com/akihiroxob/wacha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
