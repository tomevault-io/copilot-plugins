## forky

> Forky — a branching conversation tree for LLMs. Chat at any node, fork to explore different directions. Like Git for conversations.

# CLAUDE.md

## What is this?

Forky — a branching conversation tree for LLMs. Chat at any node, fork to explore different directions. Like Git for conversations.

## Tech

Bun, Next.js (App Router), TypeScript, PostgreSQL (Docker), Drizzle ORM, Anthropic SDK, Zustand, Zod.

## Setup

```bash
bun install
docker compose up -d
bun run db:migrate
bun run dev
```

## Env

```
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=postgresql://forky:forky@localhost:5432/forky
```

## Commands

- `bun run dev` — start dev server
- `bun run build` — production build
- `bun test` — run tests
- `bun run db:generate` — generate migrations from schema
- `bun run db:migrate` — run migrations
- `bun run db:studio` — open Drizzle Studio
- `bun run typecheck` — type check
- `bun run lint` — lint

## Architecture

Clean Architecture, organized by feature. Each feature has `domain/`, `application/`, `infrastructure/`, and `components/` layers. Dependency rule: inner layers never import outer layers.


## Rules

- Import features only through their `index.ts` barrel export.
- Domain and application layers are server-only. Components, hooks, and stores are client-only.
- One Zustand store per feature. Cross-feature state goes through React context.
- Route handlers and pages are thin — delegate to use cases immediately.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jimmysamportfolio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
