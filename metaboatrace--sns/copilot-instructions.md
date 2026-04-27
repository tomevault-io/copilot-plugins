## sns

> This document outlines the technical decisions and implementation guidelines for the METABOATRACE SNS project.

# Project Implementation Guidelines

This document outlines the technical decisions and implementation guidelines for the METABOATRACE SNS project.

## Core Architecture

### Framework & Routing

- **Next.js App Router** - Use App Router exclusively (no Pages Router)
- **Server Components by Default** - Prefer Server Components
- **Route Groups** - Use route groups like `(public)`, `(protected)` for organization without affecting URLs

### Styling

- **Tailwind CSS** - Utility-first CSS framework
- **No Hardcoded Colors** - Use CSS variable-based semantic tokens instead of direct color classes (e.g., `text-foreground` instead of `text-gray-800`, `bg-secondary` instead of `bg-gray-100`)
- **Semantic Tokens** - `bg-background`, `bg-secondary`, `bg-card`, `bg-primary`, `bg-destructive`, `text-foreground`, `text-muted-foreground`, `text-primary-foreground`, `text-destructive-foreground`, `border-border`

### Internationalization

- **Japanese only** - No multi-language support
- **Labels via JSON** - UI labels are defined in `src/app/admin/_lib/labels.json` and accessed via `getLabel()` utility. Do NOT hardcode Japanese strings in views.

## Testing

### Framework

- **Vitest** - Unit and integration testing framework

### Test File Organization

- **Unit Tests** - Co-located with source files in `__tests__/` directories using `.test.ts` or `.test.tsx` extension
- **Naming** - Test files mirror the source file name (e.g., `banUser.ts` → `__tests__/banUser.test.ts`)

## Code Quality

### Import Path Convention

- **Same Directory** - Use relative imports: `import { Foo } from './Foo'`
- **Parent Directory** - Use relative imports: `import { Bar } from '../Bar'`
- **Two or More Levels Up** - Use absolute imports with `@` alias: `import { Baz } from '@/lib/db'`

### Server Actions (`"use server"` files) Convention

- **Only async function declarations may be exported** — Next.js requires that every export in a `"use server"` file is an async function.
- **Do NOT `export type` from `"use server"` files** — While TypeScript erases type exports at build time, the Next.js Server Actions loader treats them as runtime values, causing `ReferenceError: <TypeName> is not defined`. Import types directly from their source module in consuming files instead.

```typescript
// BAD — causes ReferenceError at runtime
'use server';
import { type ActionResult, withRateLimit } from '@/lib/actions';
export type { ActionResult }; // DO NOT DO THIS

// GOOD — no re-export of types
'use server';
import { type ActionResult, withRateLimit } from '@/lib/actions';
// Consumers import ActionResult directly from '@/lib/actions'
```

## Database Migration

- **Always use `pnpm db:run-migrate`** — This runs `scripts/migrate.ts`, which executes Drizzle migrations and then applies Supabase-specific SQL in Supabase environments (both local and production).
- **Do NOT use `drizzle-kit push`** — `push` bypasses migration tracking (`drizzle.__drizzle_migrations`) and directly syncs the schema to the DB. This causes the migration journal and actual DB state to diverge, leading to errors on subsequent `migrate` runs.
- **Schema changes workflow**: Edit `src/lib/db/schema.ts` → run `npx drizzle-kit generate --name=<migration_name>` → run `pnpm db:run-migrate`
- **Always specify `--name` when generating migrations** — Running `drizzle-kit generate` without `--name` produces opaque filenames. Always provide a descriptive name so the migration's purpose is clear from the filename alone.
  - Use **snake_case** and keep the name concise (e.g., `create_moderation_actions`, `add_metadata_to_activity_log`)
  - The prefix is configured as **timestamp format** (`20260326195535_create_moderation_actions.sql`) in `drizzle.config.ts` to avoid conflicts between branches. Do not change this to sequential index format.
  ```bash
  npx drizzle-kit generate --name=create_moderation_actions
  ```
- **Migration file structure**:
  - `drizzle/*.sql` + `drizzle/meta/` — Drizzle-managed migrations (auto-generated, tracked by journal)
  - `drizzle/supabase/` — Supabase-specific SQL (RLS, auth hooks, permissions). Applied by `migrate.ts` in Supabase environments (detected by presence of `supabase_auth_admin` role).
- **`action` columns are varchar, not pgEnum** — avoids ALTER TYPE migrations for new action types.

## Local Development Setup

### Supabase Local

Local development uses [Supabase CLI](https://supabase.com/docs/guides/local-development) for PostgreSQL, Auth, Storage, Studio, and Mailpit (email testing) in Docker containers.

```bash
# Start Supabase local
npx supabase start

# Run migrations
pnpm db:run-migrate

# Stop Supabase local
npx supabase stop
```

- **Supabase Studio**: http://127.0.0.1:54323
- **Mailpit (email testing)**: http://127.0.0.1:54324
- **PostgreSQL**: `postgresql://postgres:postgres@127.0.0.1:54322/postgres`
- **API**: http://127.0.0.1:54321

### Environment Variables

- `NEXT_PUBLIC_SUPABASE_URL` — Supabase API URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` — Supabase publishable (anon) key
- `SUPABASE_SERVICE_ROLE_KEY` — Supabase secret (service_role) key. **Server-side only**, never prefix with `NEXT_PUBLIC_`
- `DATABASE_URL` — PostgreSQL connection string

## Moderation & Audit Architecture

### Design Pattern: Event Log with Materialized State

- **State flag** on `profiles` table (`bannedAt`) — enables O(1) status checks via `isUserBanned()`.
- **Event log** in `moderation_actions` table — immutable, append-only audit trail recording all admin operations with full context (who acted, on what, why, when). Corrections are recorded as new events (e.g., `unban` after `ban`), never as updates to existing records.

### Activity Log

- **`user_activity_log` table** — records user actions (login, etc.) for operational monitoring.
- **`logActivityEvent()` utility** (`src/lib/activity-log.ts`) — fire-and-forget pattern. Failures are logged via `console.error` but never throw, so activity logging never breaks the main action.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/metaboatrace) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
