## client-time-tracker

> Full-stack time tracking application for client billing. Monorepo using pnpm workspaces.

# Client Time Tracker

## Project Overview

Full-stack time tracking application for client billing. Monorepo using pnpm workspaces.

## Environment Variables

Defined in `.env.local` at project root:

- `JWT_SECRET` - JWT signing secret
- `API_PORT` - Backend port (default: 3001)
- `NEXT_PUBLIC_API_URL` - API base URL for the frontend (default: http://localhost:3001)

## Architecture Notes

### Authentication Flow

1. Frontend POSTs to `/api/auth/login` with `{ username, password }`
2. Backend validates credentials, returns `{ token, user }`
3. Token stored in localStorage (`ctt_token`), user in (`ctt_user`)
4. All subsequent API calls include `Authorization: Bearer <token>` header
5. Protected frontend routes use `(app)/layout.tsx` which checks `isAuthenticated()`

### Hono Middleware Order (server `index.ts`)

The middleware registration order matters. Auth routes (`/api/auth/*`) handle their own
authentication internally — the global `requireAuth` middleware explicitly skips them.
Other routes under `/api/*` go through the global auth middleware.

CORS is configured to allow any origin (for LAN access) with credentials support.

### Database

Uses PGlite (embedded Postgres via WASM). Schema defined with Drizzle ORM in
`packages/shared/src/schema.ts`. No external database server needed.

**NEVER delete, drop, recreate, or reset the database or its tables without explicitly telling the user first and getting confirmation.** The database contains real client/billing data. Schema changes must always use additive migrations (`ALTER TABLE ... ADD COLUMN IF NOT EXISTS`), never destructive ones. Do not run `pnpm db:seed` unless the user specifically asks — it may overwrite existing data.

### Supabase Sync

Optional cloud sync to a Supabase PostgreSQL database for team collaboration.

- **Config**: Stored in `data/supabase-config.json` (URL, database URL, API keys)
- **Config export/import**: `POST /api/supabase/config/export` and `/config/import` — AES-256-CBC encrypted config strings (`CTT:...`) for easy sharing between installations
- **Sync engine**: Bidirectional push/pull with changelog tracking and timestamp-based conflict resolution (`packages/shared/src/db/sync-engine.ts`)
- **Scheduler**: Background sync every 30s when enabled (`packages/shared/src/db/sync-scheduler.ts`)
- **Sidebar indicator**: Shows sync state (connected/syncing/offline/error) with manual sync button for admin/partner users
- **Routes**: `packages/server/src/routes/supabase.ts` — config, test-connection, setup-schema, sync, initial-sync

### Error Handling

- **Global error handler**: `app.onError()` in `packages/server/src/index.ts` catches unhandled exceptions and returns structured `{ error: message }` JSON
- **Time entry creation**: Try-catch with descriptive messages for FK violations ("Invalid reference: one of the selected items...") instead of generic 500s

### Rate Tiers & Time Entry Rates

- Time entry dialog uses a free-form rate input with clickable rate tier suggestion chips
- Auto-defaults rate from: client's `defaultHourlyRate` → global `baseHourlyRate` setting → first active rate tier
- Custom rates auto-create new rate tiers on save via `resolveRateTierId()`

---
> Source: [mrdatawolf/Client-Time-Tracker](https://github.com/mrdatawolf/Client-Time-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
