## 123impact

> 123impact — volunteer scheduling and event management SaaS for nonprofits. Organizations create events with shifts; volunteers register publicly (no account required), check in via QR codes, and receive automated emails. Paid tiers gate multi-day events, custom domains, and external platform sync.

# CLAUDE.md

123impact — volunteer scheduling and event management SaaS for nonprofits. Organizations create events with shifts; volunteers register publicly (no account required), check in via QR codes, and receive automated emails. Paid tiers gate multi-day events, custom domains, and external platform sync.

## Stack

- Next.js 16 (App Router) · React 19 · TypeScript 5 (strict)
- Tailwind CSS 4 + shadcn/ui (Radix primitives in `components/ui/`)
- Supabase: auth + Postgres with RLS (`supabase/migrations/`, 22 migrations)
- Stripe subscriptions (monthly/annual), MailerSend email
- Deployed on Vercel; cron schedules in `vercel.json`
- Using Cron-job.org to schedule additional crons
- Google places for location verification

## Commands

```bash
npm run dev          # dev server on :3000
npm run build        # production build
npm run lint         # eslint
npx tsc --noEmit     # type check
```

There is **no test suite** — verify changes by running the app and type-checking.

## Layout

- `app/` — public pages (`/`, `/login`, `/optout`, `/events/[eventId]/signup`), volunteer `/dashboard`, org `/admin/*` (events, volunteers, messages, analytics, settings, organizations), `/auth/callback` + `/auth/verify`
- `app/api/` — feature routes (`billing/`, `events/`, `messages/`, `qr/`, `checkin/`, `custom-domain/`), `cron/` jobs (registration-digest, billing-check, process-optouts, sync-external-events, send-scheduled-emails), and a large catch-all at `api/[[...path]]/route.ts`
- `lib/` — `supabase.ts` (browser client singleton), `email.ts` + `email-templates.ts` (MailerSend), `stripe.ts`, `platforms/` (Eventbrite/Luma fetch + `sync.ts`), `database.types.ts` (generated Supabase types)
- `contexts/OrganizationContext` — current-org state for admin UI
- `middleware.ts` — custom-domain host detection and admin auth protection

## Domain model

organizations → events → shifts → registrations, with waitlists and shiftless events supported. Roles: org owner / admin / member, plus per-event admins invited by token. QR check-in (`qr_code_instances` → `qr_scan_events` → `check_ins`). Custom signup domains per org (`org_custom_domains`). Global email opt-out list filtered before every send.

## Conventions (these prevent known regressions)

- **Supabase in components**: always import the `getBrowserClient()` singleton from `lib/supabase.ts`. Never call `createBrowserClient()` directly in a component, and don't create parallel Supabase client modules — `lib/supabase.ts` is the only browser-client source.
- **Auth state listeners**: react only to the `SIGNED_OUT` event.
- **OAuth redirects**: code exchange goes through `/dashboard` with a whitelisted `next` param (`/admin/*`, `/invite/`, `/event-invite/`, `/events/`). Extend the whitelist; don't bypass it.
- **Service-role key**: server routes only — never in client-reachable code.
- **File extensions**: codebase has legacy `.jsx`; all new files should be `.tsx`.
- **Paid-feature gating**: check the org's plan before exposing multi-day events, custom domains, or platform sync.

---
> Source: [JoshuaParker03u/123impact](https://github.com/JoshuaParker03u/123impact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
