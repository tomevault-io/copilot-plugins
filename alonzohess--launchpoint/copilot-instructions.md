## launchpoint

> LaunchPoint is a membership CRM web application for Launch Point RC, a non-profit radio control club. It tracks members, dues payments, membership status, and renewal dates. The club focuses on three RC disciplines: Track, Crawler, and Drifter.

# CLAUDE.md

## Project Overview

LaunchPoint is a membership CRM web application for Launch Point RC, a non-profit radio control club. It tracks members, dues payments, membership status, and renewal dates. The club focuses on three RC disciplines: Track, Crawler, and Drifter.

Event sponsored by Global RC Innovations Incorporated.

## Tech Stack

- **Framework:** Next.js 16 (App Router) + TypeScript
- **Styling:** Tailwind CSS v4 + shadcn/ui components
- **Database/Auth:** Supabase (Postgres + Auth + RLS)
- **Deployment:** VPS with Nginx + PM2 + Let's Encrypt SSL (see `docs/DEPLOYMENT.md`)

## Build and Run

```bash
# Install dependencies
npm install

# Run dev server
npm run dev

# Build for production
npm run build
```

## Architecture

- `src/app/login/` — Public login page (email/password via Supabase Auth) with club logo
- `src/app/(authenticated)/` — Route group requiring auth, wraps all protected pages
- `src/app/(authenticated)/layout.tsx` — Fetches user profile/role, provides via React context
- `src/app/(authenticated)/dashboard/` — Summary stats, expiring members, recent payments
- `src/app/(authenticated)/members/` — Searchable/filterable member list, CRUD
- `src/app/(authenticated)/members/[id]/` — Member detail with payment history
- `src/app/(authenticated)/members/[id]/edit/` — Edit member + record payment forms
- `src/app/(authenticated)/members/new/` — Add new member form
- `src/app/(authenticated)/payments/` — Payment history, YTD revenue
- `src/app/(authenticated)/settings/` — Club config, user role management (admin only)
- `src/app/(authenticated)/actions.ts` — Server Actions for all CRUD operations
- `src/components/app-nav.tsx` — Responsive nav with logo, links, user info, mobile hamburger
- `src/components/member-form.tsx` — Add/edit member form
- `src/components/payment-form.tsx` — Record payment form (monthly/annual/custom selector)
- `src/components/status-badge.tsx` — Colored badge for active/expired/pending
- `src/components/role-gate.tsx` — Conditionally renders children based on user role
- `src/components/ui/` — shadcn/ui primitives (button, card, table, input, etc.)
- `src/lib/supabase/client.ts` — Browser-side Supabase client
- `src/lib/supabase/server.ts` — Server-side Supabase client (cookie-based sessions)
- `src/lib/supabase/types.ts` — TypeScript types for all database tables
- `src/lib/constants.ts` — Membership types, payment methods, formatters (currency, dates)
- `src/hooks/use-user.ts` — React context for current user profile and role
- `src/middleware.ts` — Session refresh, auth redirect for unauthenticated users
- `supabase/migrations/` — SQL schema (tables, RLS policies, functions, indexes)
- `public/logo.png` — Club logo (used on login page and nav bar)

## Key Conventions

- Money stored as integer cents (divide by 100 for display)
- Member status driven by `expiration_date` (auto-expire via `expire_memberships()` function)
- Members are roster entries; Users are app logins (separate concepts — most club members don't need a login)
- RLS enforces role-based access at the database level (defense in depth)
- Server Actions check admin role before writes
- Responsive design: table on desktop, card layout on mobile/iPad
- Membership types: Track, Crawler, Drifter

## Roles

- **admin** — Full CRUD on members, payments, settings, user management
- **viewer** — Read-only access to roster and payments

## Database

Tables: `profiles`, `members`, `payments`, `settings`

- `profiles` — Extends `auth.users` with role (admin/viewer) and display name. Auto-created on signup via trigger.
- `members` — Club roster with contact info, membership type, status, join/expiration dates, notes.
- `payments` — Dues payment history linked to members. Amount in cents, tracks payment method and period covered.
- `settings` — Key-value club configuration (monthly dues, annual dues, reminder days, club name).

See `supabase/migrations/001_initial_schema.sql` for full schema with RLS policies.

## Setup

1. Create a Supabase project at supabase.com
2. Copy `.env.local.example` to `.env.local` and fill in `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
3. Run `supabase/migrations/001_initial_schema.sql` in the Supabase SQL Editor
4. Create a user via Supabase Auth dashboard (Authentication > Users > Add User)
5. Promote to admin: `UPDATE profiles SET role = 'admin' WHERE id = '<user-id>';`
6. Run `npm install && npm run dev`

## Dues Structure

- Monthly and annual dues amounts are configurable in Settings (admin only)
- Payment form offers Monthly, Annual, or Custom payment types
- Monthly/Annual auto-fills the amount and calculates the coverage period
- Recording a payment with a period end date auto-updates the member's status to active and sets their expiration date

## Deployment

- **Domain:** alonzohess.foo
- **Server:** Linode VPS (45.79.194.140), Ubuntu
- **Stack:** Nginx reverse proxy → PM2 → Next.js on port 3000
- **SSL:** Let's Encrypt via Certbot (auto-renews)
- See `docs/DEPLOYMENT.md` for full setup and update instructions

## Network Access

Dev server configured to allow access from `192.168.1.160` (see `next.config.ts` `allowedDevOrigins`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alonzohess)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alonzohess)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
