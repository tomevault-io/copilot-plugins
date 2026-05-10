## xports

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**Xports** is a self-hosted X/Twitter bookmarks organizer. It imports bookmarks exported via a browser console script, stores them in PostgreSQL, supports AI-assisted categorization (OpenAI or Ollama), and serves a React SPA over LAN. No authentication — open by design.

## Monorepo Structure

pnpm workspaces with three packages:

| Package | Path | Role |
|---------|------|------|
| `api-server` | `artifacts/api-server/` | Express 5 backend, serves API + frontend static files |
| `twitter-bookmarks` | `artifacts/twitter-bookmarks/` | React 19 + Vite SPA |
| `db` | `lib/db/` | Drizzle ORM schema + migrations |
| `api-client-react` | `lib/api-client-react/` | React Query hooks used by the frontend |
| `shared` | `lib/shared/` | Zod schemas + TypeScript types shared across packages |

## Common Commands

```bash
# Development (runs api-server + frontend in parallel)
pnpm dev

# Build everything
pnpm build

# TypeScript check across all packages
pnpm typecheck

# Database migrations
pnpm migrate

# Run only api-server
pnpm --filter api-server dev

# Run only frontend
pnpm --filter twitter-bookmarks dev

# Docker (production)
docker compose up --build
```

## Architecture

### Request Flow

1. Browser → Vite dev proxy `/api` → Express backend (`:3000`)
2. Active profile ID is stored in `localStorage` and sent as `x-xport-profile-id` header on every API request (set in `lib/api-client-react/src/client.ts`)
3. Routes scope all queries by `profileId`

### Backend (`artifacts/api-server/src/`)

- `app.ts` — Express app setup, static file serving
- `config.ts` — env parsing (port, AI providers, DB URL)
- `routes/` — REST endpoints: `profiles`, `bookmarks`, `categories`, `ai`, `stats`
- `services/ai.ts` — pluggable AI categorization (OpenAI SDK pointing at OpenAI or Ollama)
- `profiles.ts` — upsert helper called on import

### Frontend (`artifacts/twitter-bookmarks/src/`)

- Routing via **Wouter**
- Server state via **React Query** (TanStack Query)
- Active profile stored in `localStorage`, exposed via `useActiveProfile` context hook
- UI: **Radix UI** + **shadcn/ui** components + **Tailwind CSS**
- Pages: `Dashboard`, `Bookmarks`, `Categories`, `Import`, `Export`

### Database (`lib/db/src/`)

- **Drizzle ORM** on **PostgreSQL 17**
- Schema: `profiles`, `bookmarks`, `categories`
- Bookmarks are unique on `(profileId, tweetId)`; cascade delete on profile removal
- Migrations in `lib/db/src/migrations/`

### AI Integration

Configured via env vars:
- `AI_ENABLED_PROVIDERS=ollama,openai`
- `OLLAMA_BASE_URL`, `OLLAMA_MODEL`
- `OPENAI_API_KEY`, `OPENAI_BASE_URL`, `OPENAI_MODEL`

Both providers use the OpenAI SDK. Ollama is reached via `host.docker.internal:11434/v1`.

### Data Import

The browser-side collector script (`lib/bookmarks-collector-script.ts`) is pasted into the X/Twitter console to scrape bookmarks as JSON. That JSON is then uploaded via the `/import` page. The server auto-creates/updates the profile on import.

## Environment

Copy `.env.example` to `.env`. Key vars:

```
DATABASE_URL=postgresql://...
AI_ENABLED_PROVIDERS=ollama,openai
VITE_API_PROXY_TARGET=http://127.0.0.1:3000
APP_PORT=3000
```

## Collaboration Preferences

- Respond in **pt-BR**
- Avoid large refactors without clear need
- Prefer local/self-hosted solutions

---
> Source: [felipedruzian/Xports](https://github.com/felipedruzian/Xports) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
