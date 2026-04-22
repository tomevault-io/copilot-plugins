## momentto

> Momentto is a travel countdown PWA (Progressive Web App) built with Next.js. Users create countdowns to upcoming trips, share them publicly, and receive push notifications. The app supports offline use via a service worker and syncs data to Supabase.

# CLAUDE.md

## Project Overview

Momentto is a travel countdown PWA (Progressive Web App) built with Next.js. Users create countdowns to upcoming trips, share them publicly, and receive push notifications. The app supports offline use via a service worker and syncs data to Supabase.

## Commands

```bash
npm run dev      # Start local dev server (http://localhost:3000)
npm run build    # Production build
npm run start    # Run production server from .next/
npm run lint     # Run Next.js lint rules
```

Use `pnpm` or `yarn` interchangeably with the same script names.

## Architecture

### Tech Stack

- **Framework**: Next.js 16 (App Router) with React 19
- **Language**: TypeScript (strict)
- **Styling**: Tailwind CSS v4
- **UI primitives**: Radix UI (shadcn/ui convention via `components/ui/`)
- **Animations**: Motion (Framer Motion v11)
- **Backend**: Supabase (Postgres + Auth + Storage)
- **Push notifications**: Web Push + VAPID keys
- **i18n**: next-intl (locales: `en`, `pt-BR`)
- **PWA**: next-pwa (service worker generated into `public/sw.js`)
- **Deployment**: Vercel

### Directory Structure

```
app/                    Next.js App Router
  [locale]/             Locale-prefixed routes (en / pt-BR)
    page.tsx            Landing page
    app/page.tsx        Main app (countdown list)
    login/              Auth flow
    c/[shareId]/        Public shared countdown view
    changelog/          Changelog page
  auth/callback/        Supabase OAuth callback (no locale prefix)
  actions.ts            Server Actions
  layout.tsx            Root layout
components/             Feature and UI components
  ui/                   shadcn/ui primitives (do not modify style conventions)
lib/
  supabase/
    client.ts           Browser Supabase client
    server.ts           Server Supabase client (cookies-based)
  db.ts                 IndexedDB helpers (offline storage via idb)
  sync.ts               Sync logic between IndexedDB and Supabase
  countdowns.ts         Countdown CRUD helpers
  types.ts              Shared TypeScript types
  analytics.ts          GA4 event helpers
  *-context.tsx         React contexts (auth, accent color, connectivity, active countdown)
hooks/                  Custom React hooks
supabase/
  schema.sql            Database schema
middleware.ts           Auth session refresh + next-intl locale routing
```

### Key Patterns

- **Locale routing**: All user-facing pages live under `app/[locale]/`. The middleware enforces locale prefixes, migrates legacy `momentto-language` cookies, and redirects legacy `/c/[shareId]` links.
- **Auth**: Supabase Auth with server-side session refresh in middleware. Use `lib/supabase/server.ts` in Server Components and Server Actions; use `lib/supabase/client.ts` in Client Components.
- **Offline-first**: IndexedDB (`lib/db.ts`) stores countdowns locally. `lib/sync.ts` reconciles local state with Supabase when online.
- **Push notifications**: VAPID-based Web Push. Subscription management is in `components/push-notification-manager.tsx`.
- **Server Actions**: Defined in `app/actions.ts`; used for mutations that require server-side Supabase access.

## Environment Variables

Copy `.env.local.example` to `.env.local` and fill in:

| Variable | Required | Description |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Yes | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Yes | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | Service role key (server only) |
| `NEXT_PUBLIC_VAPID_PUBLIC_KEY` | Yes | VAPID public key for Web Push |
| `VAPID_PRIVATE_KEY` | Yes | VAPID private key (server only) |
| `NEXT_PUBLIC_GTM_ID` | No | Google Tag Manager container ID |
| `NEXT_PUBLIC_GA_MEASUREMENT_ID` | No | GA4 Measurement ID |

Never commit `.env*` files.

## Coding Conventions

- 2-space indentation; match existing file formatting.
- PascalCase for components (`CountdownDisplay.tsx`), camelCase for hooks (`useCountdown`) and utilities.
- Tailwind utility classes are the only styling mechanism — no CSS Modules or inline styles.
- Keep `components/ui/` components close to shadcn/ui defaults; customise via `className` props, not by editing primitives directly.
- Prefer Server Components by default; add `"use client"` only when necessary (event handlers, browser APIs, context consumers).
- Use `next-intl` `useTranslations` / `getTranslations` for all user-visible strings — no hardcoded copy outside translation files.

## Git Conventions

- Commit prefix: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`
- Keep commits scoped: `feat: add expiry option to share modal`
- PRs should include a summary, test notes, and screenshots for UI changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carrijoga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
