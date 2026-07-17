## truckmates-com

> Handles money, driver compliance, and customer data. For multi-file or schema/billing/auth changes,

# TruckMates — Claude Code Project Guide

TruckMates (truckmates.com) is a **multi-tenant logistics SaaS** for carriers and fleets:
dispatch, load lifecycle, finance, compliance (IFTA/ELD/DVIR/HOS), CRM, reporting, and an AI
automation agent. This is a **production app on Vercel** — treat changes accordingly.

## Stack
- **Next.js 16** App Router + **React 19** + **TypeScript** (strict)
- **Supabase**: Auth + Postgres + **RLS** (tenant isolation is enforced at the DB layer)
- **Tailwind v4** + **shadcn/Radix** UI
- Billing: **Stripe + Paddle + PayPal** · Email **Resend** · SMS **Twilio** · Push **Firebase**
- AI: **Claude** via `lib/ai/client` (`callClaude`) · Observability: **Sentry**, **PostHog**, **Vercel Analytics**

## Where things live
- `app/` — App Router routes (`app/page.tsx`, `app/dashboard/...`, `app/api/...`)
- `app/actions/` — **server actions** (~180 files); the primary data-mutation layer
- `app/api/` — route handlers: `webhooks/`, `cron/`, `v1/` (public API), `ai/`
- `lib/` — domain logic: `ai/` (agent), `supabase/` (clients), `auth/`, `billing/`, `eld/`, `hos/`, plan gating, compliance
- `supabase/migrations/` — SQL migrations (RLS policies live here)
- `components/` — shared UI · `middleware.ts` — auth + route + subscription gating
- `truckmates-eld-mobile/` — separate React Native ELD app (out of scope for web tasks)

There is **no `src/` directory.**

## Non-negotiable conventions

### 1. Multi-tenant isolation (most important)
Every business row is scoped by `company_id`. Two Supabase clients:
- **`lib/supabase/server.ts` → `createClient()`** — authenticated, **RLS-enforced**. Prefer this whenever there is a user session.
- **`lib/supabase/admin.ts` → `createAdminClient()`** — service role, **bypasses RLS**. Only in webhooks/cron/system code. **Every query MUST `.eq("company_id", companyId)`** — a missing filter is a cross-tenant data leak.

→ Use the **`tenant-isolation`** skill before writing any data access.

### 2. Auth in server actions
Identity comes from `getCachedAuthContext()` (`lib/auth/server.ts`) → `{ userId, companyId, user.role, error }`.
Roles are read from the **database**, never from JWT metadata. Return `{ data, error }` with JSON-serializable values only.

### 3. Load status is a state machine
Use `lib/load-status.ts` (`parseLoadStatus`). Don't invent transitions (e.g. pending → delivered skipping in_transit).

### 4. AI agent changes
Adding/altering an automation touches several coordinated files. → Use the **`add-ai-automation`** skill; don't wire it by hand.

### 5. Schema changes
→ Use the **`supabase-migration`** skill. Every new tenant table needs `company_id` + RLS policies.

### 6. UI
Reuse existing shadcn/Radix components and Tailwind tokens. Don't introduce new component libraries.

## Commands
- `npm run dev` → http://127.0.0.1:3000 (run from repo root; webpack)
- `npm run test:unit` (vitest) · `npm run lint` · `npm run build`

## Safety
Handles money, driver compliance, and customer data. For multi-file or schema/billing/auth changes,
plan before editing (plan mode). Never run destructive SQL or touch another tenant's data.

---
> Source: [IBR0521/TruckMates.com](https://github.com/IBR0521/TruckMates.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
