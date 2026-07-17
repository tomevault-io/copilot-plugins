## swissperiences

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swissperiences is a membership-based luxury travel platform for Switzerland. It's a Vite + React + TypeScript SPA with a Supabase backend (PostgreSQL, Edge Functions, Auth) and Vercel serverless API routes. Payments via Stripe, emails via Resend, rate limiting via Upstash Redis.

## Development Commands

```bash
# Full local dev (frontend + API) — recommended
vercel dev                    # Serves everything on port 3000

# Frontend only (no API routes)
npm run dev                   # Vite dev server on port 8080

# Two-terminal setup (advanced)
vercel dev --listen 3001      # Terminal 1: API server
npm run dev                   # Terminal 2: Frontend (proxies /api/* to 3001)

# Build & lint
npm run build                 # Production build
npm run lint                  # ESLint (flat config)
```

**Important**: `npm run dev` does NOT serve `/api/*` routes. Use `vercel dev` for full functionality.

No test framework is configured.

## Architecture

### Routing (src/App.tsx)
- React Router v6 with lazy-loaded routes via `React.lazy` + `<Suspense>`
- Dual route pattern: bare paths (`/experiences`) + language-prefixed (`/:lang/experiences`)
- `<LanguageWrapper>` syncs URL `:lang` param with i18next (en/pt)
- `<AuthGuard>` wraps protected routes, checks auth + membership via `get_or_create_member` RPC
  - `requireAdmin` flag checks `is_admin` RPC

### Auth Flow
- Two methods: Google OAuth (primary) + Email/Password (expandable form on `/login`)
- OAuth: `/login` → Google → `/auth/callback` → `onAuthStateChange` listener → `get_or_create_member` RPC → route by membership status
- Email: `/login` → `signInWithPassword` → same RPC routing
- Membership routing: `active` → `/members`, `pending` → `/pending-approval`, `no_application` → `/request-access`

### Two Serverless Layers
1. **Vercel API routes** (`/api/`): TypeScript functions for checkout, emails, webhooks
   - `api/create-checkout-session.ts` — Stripe checkout
   - `api/webhooks/stripe.ts` — Stripe webhook handler
   - `api/send-inquiry-email.ts`, `api/send-guest-inquiry.ts`, `api/send-waitlist-email.ts` — Resend emails
   - `api/lib/rate-limit.ts` — Upstash Redis rate limiting
2. **Supabase Edge Functions** (`/supabase/functions/`): Deno runtime
   - `newsletter-signup` — Resend audience + welcome email (called with hardcoded JWT in `src/lib/newsletter.ts`)
   - `notify-application` — DB trigger fires on `membership_applications` INSERT
   - `send-approval-email`, `send-booking-confirmation` — Admin-triggered emails
   - `create-checkout`, `stripe-webhook` — Stripe integration

### Frontend Patterns
- **UI**: shadcn/ui components (`src/components/ui/`) with Radix primitives
- **Styling**: Tailwind CSS, dark-mode-first, custom Swiss-luxury design tokens in `src/index.css`
- **Fonts**: Inter (body), Lora (serif headlines)
- **State**: TanStack React Query for server state, React Hook Form + Zod for forms
- **Toasts**: Sonner (`sonner` package)
- **Analytics**: Plausible (privacy-friendly, script in index.html)
- **Path alias**: `@/*` maps to `src/*`

### Data
- Static data files in `src/data/` (cities, packages, journals, retreats)
- Supabase types auto-generated in `src/integrations/supabase/types.ts`
- i18n translations loaded from `/public/locales/{en,pt}/`

### Database (Supabase)
Key tables: `membership_applications`, `members`, `bookings`, `blocked_dates`, `admin_emails`, `waitlist`, `corporate_inquiries`

Migrations in `supabase/migrations/`. Always use `DROP POLICY IF EXISTS` before `CREATE POLICY` for idempotent migrations.

### Newsletter Signup Quirk
`src/lib/newsletter.ts` uses direct `fetch()` with a hardcoded Supabase anon JWT key instead of the Supabase client. This is intentional — the `sb_publishable_*` key from Vercel env vars works with supabase-js but NOT as a raw Authorization header for edge function calls.

## Deployment
- **Frontend + API**: Vercel (Hobby plan, 100 deploys/day shared across team)
- **Backend**: Supabase project `rhoxismvcalqppbnndew` (West EU)
- `vercel.json` configures security headers (CSP, HSTS), CORS for `/api/*`, and SPA rewrites

---
> Source: [swissperiences/swissperiences](https://github.com/swissperiences/swissperiences) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
