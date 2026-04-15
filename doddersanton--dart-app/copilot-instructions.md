## dart-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server with Turbopack
npm run build        # Build production bundle
npm run lint         # Run ESLint
npm run db:generate  # Generate Drizzle migrations from schema changes
npm run db:push      # Push schema to PostgreSQL (Neon serverless)
```

There are no tests in this codebase.

## Architecture Overview

This is a **Next.js 15 App Router** full-stack app for managing a darts league team — tracking players, fines, fixtures, game results, payments, and subscriptions.

### Key Technologies
- **Auth:** Clerk (middleware protects `/fines`, `/players`, `/teams`, `/fixtures`, `/dashboard`, `/subscriptions`, `/reports`, `/settings`)
- **Database:** PostgreSQL via Neon serverless + **Drizzle ORM** (schema in `server/schema.ts`)
- **Server Actions:** `next-safe-action` with Zod validation — all mutations go through `server/actions/`
- **Payments:** Stripe (payment intents)
- **File Uploads:** UploadThing (player images)
- **UI:** shadcn/ui (New York style, neutral base) + Tailwind CSS 4 + Recharts for data viz

### Data Flow
1. Client forms submit via `next-safe-action` server actions
2. Actions validate with Zod schemas (defined in `/types/`)
3. Actions run Drizzle ORM queries against Neon PostgreSQL
4. `revalidatePath()` triggers ISR cache invalidation
5. Success/error returned to client; Sonner toasts display feedback

### Directory Structure

| Path | Purpose |
|------|---------|
| `app/` | Next.js App Router pages and layouts |
| `server/schema.ts` | Single source of truth for all Drizzle table definitions |
| `server/actions/` | ~44 server actions (one file per operation) |
| `server/migrations/` | Auto-generated SQL migrations (do not edit manually) |
| `components/ui/` | shadcn/ui primitives |
| `components/fines/`, `players/`, `fixtures/` | Domain-specific components |
| `types/` | Zod validation schemas (one file per domain entity) |
| `lib/` | Stripe client, utility functions |
| `hooks/` | Custom React hooks |

### Server Actions Pattern

All actions in `server/actions/` follow this structure:

```typescript
"use server"
import { createSafeActionClient } from "next-safe-action"

const actionClient = createSafeActionClient()

export const actionName = actionClient
  .schema(zodSchema)
  .action(async ({ parsedInput }) => {
    try {
      // Drizzle ORM query
      revalidatePath("/some-path")
      return { success: "message" }
    } catch (error) {
      return { error: JSON.stringify(error) }
    }
  })
```

### Database Schema (Key Tables)
- **players** — player records (name, nickname, team, image)
- **fines** — fine type definitions (title, amount)
- **playerFines** — issued fines with payment status, game context
- **payments** — payment records linked to players
- **subscriptions** — membership periods per player
- **fixtures** — match definitions (teams, dates, location, season)
- **games** — individual game results within fixtures
- **rounds** — detailed round/leg scoring within games
- **gamePlayers** — game ↔ player association
- **locations** — match venues
- **seasons** — league seasons (start/end dates)

### Environment Variables
Required in `.env.local`:
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` / `CLERK_SECRET_KEY` — Clerk auth
- `NEXT_PUBLIC_DATABASE_URL` — Neon PostgreSQL connection string
- `NEXT_PUBLIC_PUBLISH_KEY` / `STRIPE_SECRET` — Stripe

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DoddersAnton)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DoddersAnton)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
