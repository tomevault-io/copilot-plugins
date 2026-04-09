## clarity

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Clarity is a self-hosted, single-user personal productivity platform combining audio transcription, rich notes, task management, journaling, bookmarks, kanban boards, and whiteboarding. It uses AI providers (OpenAI, Groq, AssemblyAI) for transcription and text refinement.

## Commands

```bash
# Development
pnpm install              # Install dependencies
pnpm dev                  # Start dev server on http://localhost:3000

# Build & Production
pnpm build                # Next.js production build
node .next/standalone/server.js  # Start production server

# Database
pnpm db:generate          # Generate Drizzle migration files after schema changes
pnpm db:migrate           # Run pending migrations (tsx lib/api/db/migrate.ts)
pnpm db:studio            # Open Drizzle Studio GUI

# Linting
pnpm lint                 # ESLint (next/core-web-vitals config)
```

No test framework is configured.

## Architecture

### Dual API Layer

The backend has two parallel API systems within the same Next.js app:

1. **Hono API** (`lib/api/`) — mounted at `/api/v1/[[...route]]` via a Next.js catch-all route. Handles core business logic: auth, notes, transcripts, providers, settings, storage, sync, AI, and public endpoints. All routes go through Hono middleware (logger, CORS, JWT auth).

2. **Next.js API routes** (`app/api/`) — handle features added later: bookmarks, tasks, journal, kanban, canvas, search, uploads, and external integrations. These use direct Drizzle queries without Hono middleware.

Both layers share the same database and schema. When adding new API endpoints, check which pattern the feature currently uses.

### Database

SQLite (better-sqlite3) with Drizzle ORM. Single file at `./data/clarity.db` (configurable via `DATABASE_PATH` env). WAL mode enabled.

- Schema: `lib/api/db/schema.ts` — 15 tables with exported TypeScript types
- Migrations: `drizzle/migrations/`
- Client: `lib/api/db/client.ts`
- Patterns: soft deletes via `deletedAt`, version tracking on notes/transcripts, JSON arrays stored as text (tags, models), timestamps as Unix integers

### Authentication

JWT-based with httpOnly cookies. System secrets (JWT key, encryption key) are auto-generated at first setup and stored in the `systemConfig` table, not env vars. Auth flow: setup → login → 15-min access token + 7-day refresh token with automatic rotation. Provider API keys are encrypted at rest with AES-256-GCM.

### Frontend

- **Routing**: Next.js App Router. Auth pages in `app/(auth)/`, protected pages in `app/(main)/` behind `AuthGuard`.
- **Components**: shadcn/ui (new-york style) built on Radix UI primitives. Located in `components/ui/`. Feature components in `components/{feature}/`.
- **State**: React hooks + Context (OnboardingContext only). No Redux/Zustand. Data fetched via `apiFetch()` wrapper in `lib/storage.ts` which handles token refresh.
- **Editor**: TipTap with extensions for markdown, mermaid diagrams, code highlighting, tables, backlinks.
- **Styling**: Tailwind CSS v4 with CSS variables defined in `app/globals.css`. Design uses 0 border-radius by default. Accent colors customizable (6 presets stored in localStorage as `clarity_accent_color`).
- **Animations**: Framer Motion. Respects `prefers-reduced-motion`.

### Key Path Aliases

`@/` maps to the project root (`tsconfig.json`). Examples: `@/components/ui/button`, `@/lib/utils`, `@/lib/api/db/schema`.

### Data Flow Pattern

Client components fetch data with `apiFetch()` from `lib/storage.ts` → hits Hono routes at `/api/v1/*` or Next.js routes at `/app/api/*` → Drizzle ORM queries SQLite → returns JSON. On 401, `apiFetch` automatically refreshes the token and retries.

### Browser Extension

A Chrome extension lives in `extension/` for quick capture. Communicates with the main app API using body-based token refresh (`/auth/refresh-token`).

## Conventions

- All interactive components use `"use client"` directive
- Heavy components (TipTap editor, Excalidraw) are dynamically imported with `ssr: false`
- `cn()` from `lib/utils.ts` (clsx + tailwind-merge) for conditional class names
- Component variants use CVA (class-variance-authority)
- IDs generated with `nanoid`
- Toast notifications via `sonner` (`toast.success()`, `toast.error()`)
- Optimistic UI updates — state is updated before API confirmation, rolled back on error
- Icons from `lucide-react`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Harsh-2002)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Harsh-2002)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
