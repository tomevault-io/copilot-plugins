## ally

> Rebuild from scratch. **Read the "Modus operandi" section before writing any code — it is the reason this repo exists.**

# ally

Rebuild from scratch. **Read the "Modus operandi" section before writing any code — it is the reason this repo exists.**

## Why we started over (read this first)

The previous version was **fully vibe-coded**: generated in large, unreviewed chunks until nobody understood it. Fixing it ended up **more expensive than starting over**, so we did. This time the non-negotiable goal is: **do not lose control of the codebase.**

## Modus operandi

Every change must stay small enough that a human keeps full understanding of it.

1. **Atomic implementations.** One coherent change per step. No sprawling multi-feature dumps.
2. **Few files per edit.** Touch the minimum. If a task needs many files, split it into steps and do them one at a time.
3. **Verifiable in the editor.** The diff must be small enough to read and confirm by eye. If it can't be reviewed in the editor, it's too big — break it down.
4. **Explain what and why.** After every change, state plainly what was done and the reasoning behind it. No silent edits.
5. **Plan before building anything non-trivial.** Architecture is decided up front (see the architect), not discovered mid-implementation.

When in doubt, prefer a smaller step and ask. Losing control is the failure mode we are explicitly engineering against.

### Control level per surface

The discipline above is not uniform — it is tightest on the backend and loosest on the landing/UI.

- **Backend (`apps/api`) — highest control.** This is where the rules reign most. The flow is: **the user requests something → the `backend-engineer` agent performs discrete, atomic tasks.** Small diffs, few files, test-first, explained. No large autonomous refactors. This is the core of the system and the part we most refuse to vibe-code.
- **`apps/web` (React SPA) — medium control.** Real review, but more latitude than the backend, especially for UI wiring.
- **Landing / UI polish (`apps/landing`, styling) — most permissive.** This is where "vibe coding" is acceptable — **not everything, but here we can afford less control.** Visual/marketing surface, low blast radius.

## The delivery flow

1. **Architecture (`software-architect`)** — called to produce architecture **plannings** (design + trade-offs + a task breakdown). These documents are the reusable input for implementation; they are approved before any code is written. Produces no code.
2. **Implementation** — `backend-engineer` and/or `frontend-engineer` execute the approved tasks in atomic steps.
3. **Review (`code-reviewer`)** — quality gate before anything is committed.
4. **`ai-advisor`** — consulted (not an implementer) whenever AI/RAG decisions are involved.

## Monorepo layout

pnpm workspace (`pnpm-workspace.yaml` → `apps/*`, `packages/*`).

```
ally/
├── apps/
│   ├── landing/   # Astro v6 — marketing/landing (static-first)
│   ├── web/       # Vite + React 18 SPA — TanStack Router/Query/Form/Table
│   └── api/       # Bun + Hono — backend API
├── packages/      # shared code (e.g. intelligence, db, contracts, audit-core, connector-core) as introduced
└── CLAUDE.md
```

## Stack

- **landing** — Astro v6, zero-JS-by-default, React islands only where needed. Node ≥22.12.
- **web** — Vite + React 18, TanStack **Router** (SPA, no SSR) + **Query** + **Form** + **Table**.
- **api** — Bun + Hono (v4), TypeScript, `bun test`.
- **Data** — Postgres + pgvector, Redis.
- **AI** — AWS Bedrock, RAG with pgvector, evals.
- **Deploy** — *to define*.
- **Contracts** — shared **Zod** schemas in `packages/contracts`, consumed by both `api` and `web`.

## Agents (`.claude/agents/`)

- **`software-architect`** — plans architecture + task breakdown before implementation. No code.
- **`backend-engineer`** — implements `apps/api` + `packages/*`, test-first (`bun test`). Highest control.
- **`frontend-engineer`** — implements `apps/web` (React/TanStack) and `apps/landing` (Astro).
- **`code-reviewer`** — quality gate; reviews the diff, does not fix.
- **`ai-advisor`** — advises on AI/RAG/model/eval decisions; does not implement.

## Skills (`.claude/skills/`)

Stack: **`hono-bun`**, **`tanstack-router`**, **`astro`**, **`zod`**.
Craft: **`tdd`**, **`design-system`**, **`frontend-design`**, **`ui-ux-pro-max`**, **`vercel-react-best-practices`**, **`web-design-guidelines`**.

## Commands

- `pnpm dev` — all apps in parallel · `pnpm dev:api` / `dev:web` / `dev:landing` — one app.
- `pnpm build` · `pnpm lint` — across apps.
- API tests: `bun test` (in `apps/api`). Web tests: Vitest.

> When asked about a library/framework/SDK, fetch current docs via Context7 before relying on memory (see the user's global rule).

---
> Source: [Tomas-Izuel/ally](https://github.com/Tomas-Izuel/ally) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
