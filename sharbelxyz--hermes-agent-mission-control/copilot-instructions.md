## hermes-agent-mission-control

> Notes for Claude Code (or similar AI coding tools) editing this repo.

# CLAUDE.md

Notes for Claude Code (or similar AI coding tools) editing this repo.

## What this is

A Mission Control dashboard for AI agents. Next.js 16 App Router, Prisma
6 on Postgres, Tailwind v4, NextAuth optional.

## Architecture in 30 seconds

- **Source of truth for agent state** = `AgentState` table in Postgres.
- **Agents self-report** by POSTing to `/api/agents/state` with a shared
  secret (`INTERNAL_API_SECRET` in env).
- **Dashboard is read-only** - pages fetch directly from Prisma.
- **No background jobs, no WebSockets, no push.** Agents heartbeat on
  their own schedule; pages re-render on navigate. Keep it simple.

## Conventions

- `"use client"` only when you need hooks/events. Default to server.
- `dynamic = "force-dynamic"` on any page or API route that reads from
  Prisma. We don't want accidentally-cached stale data.
- No em dashes in user-facing copy. Period.
- Error handling: wrap Prisma reads in `try/catch` and show an empty
  state, don't crash the dashboard.

## Data flow reference

```
Hermes Agent (Python / TS)
       │  POST /api/agents/state
       │  Authorization: Bearer $INTERNAL_API_SECRET
       ▼
  Next.js route handler
       │  prisma.agentState.upsert
       ▼
     Postgres
       ▲
       │  prisma.agentState.findMany
       │
  Dashboard page (server component)
       ▼
     Browser
```

## Where to add a new model

1. Edit `prisma/schema.prisma`
2. Run `npx prisma db push`
3. Add a server component page under `src/app/<name>/page.tsx`
4. Add a nav entry in `src/components/sidebar.tsx`
5. (Optional) Add a route under `src/app/api/<name>/route.ts` for agent
   writes, gated behind `INTERNAL_API_SECRET`.

## Where NOT to leak secrets

- Never hardcode API keys. Use `process.env.X` only.
- `.env*` files are gitignored - check `.gitignore` before committing.
- The `data/` folder is gitignored except for `.gitkeep`. If you write
  runtime state to `data/`, don't commit it.

## Forbidden add-ons

This is a template. Do NOT add:

- Sharbel-specific integrations (Polymarket, Hyperliquid, specific
  newsletter tools) without a user asking for them.
- Tauri desktop shell (explicitly stripped from the template).
- Heavy UI libraries. Tailwind is it.
- ORMs other than Prisma.

## If the user wants the full Max HQ experience

Max HQ is @sharbelxyz's private superset of this template. It has
trading PnL cards, Polymarket integration, content briefs, a longform
script pipeline, YouTube analytics. It's not shippable as a public
template. This repo is the generic scaffold; the user builds their own
personality on top.

---
> Source: [sharbelxyz/hermes-agent-mission-control](https://github.com/sharbelxyz/hermes-agent-mission-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
