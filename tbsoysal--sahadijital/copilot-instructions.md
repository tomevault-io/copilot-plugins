## sahadijital

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Development server with Turbopack
npm run build    # Production build
npm run start    # Start production server
npm run lint     # ESLint
```

No test framework is configured.

## Architecture

This is a **Next.js 16 / React 19** B2B SaaS app for managing futsal/soccer field reservations, with a Turkish-language UI and Supabase (PostgreSQL) backend.

### Layer Structure

```
components/ → hooks/ → services/ → Supabase
```

- **`src/lib/services/`** — All Supabase queries live here (`authService`, `calendarService`, `profileService`). Services return raw data; they don't manage React state.
- **`src/lib/hooks/`** — Custom React hooks consume services and manage local state. Auth hooks in `hooks/auth/`, dashboard hooks in `hooks/dashboard/`.
- **`src/context/DashboardContext.tsx`** — Global dashboard state (user, fields, selected field, calendar view, sidebar toggle) provided to all dashboard components.
- **`src/lib/schemas/`** — Zod validation schemas for all forms (login, signup, password reset, etc.).
- **`src/types/index.ts`** — Core types: `Field`, `Reservation`, `View`, `Slot`.

### Routing

Uses Next.js App Router with two route groups:
- **`(auth)`** — Login, signup, forgot/reset password with a shared auth layout.
- **`dashboard/`** — Protected area with sidebar layout and a calendar view (Daily/Weekly/Monthly) selectable per field.

### Supabase Integration

Two clients depending on context:
- `src/lib/supabase/client.ts` — Browser-side (for client components)
- `src/lib/supabase/server.ts` — Server-side (for server components / API routes)

### Path Alias

`@/*` maps to `./src/*` — use this for all internal imports.

---
> Source: [tbsoysal/sahadijital](https://github.com/tbsoysal/sahadijital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
