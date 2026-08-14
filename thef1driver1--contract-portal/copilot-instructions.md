## contract-portal

> A bilingual (ES/EN) rental contract management SaaS for Puerto Rico landlords. Landlords generate, send, and track lease agreements; tenants receive email invites to sign digitally. Includes investment analytics, expense tracking, and tax reporting (Schedule E).

# Contract Portal — Claude Code Context

## What This App Is

A bilingual (ES/EN) rental contract management SaaS for Puerto Rico landlords. Landlords generate, send, and track lease agreements; tenants receive email invites to sign digitally. Includes investment analytics, expense tracking, and tax reporting (Schedule E).

**Live stack:** Next.js 14 (App Router) · Supabase (Postgres + Auth + Storage) · Resend (email) · Twilio (SMS) · Stripe (billing) · Vercel (hosting) · Upstash Redis (rate limiting)

**Default locale:** Spanish (`es`). Middleware reads `NEXT_LOCALE` cookie; tenant-facing UI defaults to `es`.

---

## Repo Layout

```
Contract_Portal/
├── web/                  ← Next.js app (all active development happens here)
│   ├── app/              ← App Router routes + API routes
│   ├── components/       ← Shared UI components
│   ├── lib/              ← Business logic, Supabase clients, types, schemas
│   ├── messages/         ← i18n JSON (en.json, es.json)
│   ├── templates/        ← DOCX lease template
│   └── supabase/         ← Migration files + schema reference
├── supabase/             ← Root-level migrations (PR expansion)
├── PLANS/                ← Feature roadmap docs
└── CLAUDE.md             ← This file
```

---

## Route Map

### Auth (`web/app/(auth)/`)
| Route | File |
|---|---|
| `/login` | `(auth)/login/page.tsx` |
| `/signup` | `(auth)/signup/page.tsx` |
| `/forgot-password` | `(auth)/forgot-password/page.tsx` |
| `/reset-password` | `(auth)/reset-password/page.tsx` |

### Landlord Dashboard (`web/app/(dashboard)/`)
| Route | Purpose |
|---|---|
| `/dashboard` | Main overview |
| `/contracts` | Contract list |
| `/contracts/new` | Create contract |
| `/contracts/[id]` | Contract detail, actions, notifications |
| `/properties` | Property CRUD |
| `/tenants` | Tenant CRUD |
| `/expenses` | Expense tracking (tax deductions) |
| `/market` | Market analysis (Zillow data) |
| `/market/[id]` | Property market detail |
| `/watchlist` | Investment watchlist |
| `/watchlist/[id]/analyze` | Investment ROI calculator |
| `/reports/schedule-e` | IRS Schedule E PDF generation |
| `/settings/billing` | Stripe plan management |
| `/settings/managers` | Property manager invitations |
| `/settings/notifications` | SMS/email reminder rules |
| `/settings/sections` | Custom contract clause library |
| `/settings/templates` | DOCX template uploads |
| `/profile` | User profile |

### Tenant Portal (`web/app/(portal)/`)
| Route | Purpose |
|---|---|
| `/portal` | Tenant dashboard |
| `/portal/sign/[contractId]` | Digital signature flow |

### Public
| Route | Purpose |
|---|---|
| `/invite/[token]` | Tenant invite + signup |
| `/invite/manager/[token]` | Property manager invite |
| `/` | Landing page |
| `/pricing` | Pricing page |

### API (`web/app/api/`)
Key endpoints — all authenticated unless noted:
- `POST /api/contracts` — create contract
- `GET/PUT/DELETE /api/contracts/[id]`
- `POST /api/contracts/[id]/send-email`
- `POST /api/contracts/[id]/quick-sms`
- `POST /api/contracts/[id]/invite` — email invite to tenant
- `POST /api/generate` — DOCX/PDF generation (uses `docxtemplater`)
- `GET/POST /api/properties`
- `GET/POST /api/tenants`
- `GET/POST /api/expenses`
- `GET/POST /api/templates` — custom DOCX templates
- `GET/POST /api/managers` + `POST /api/managers/invite/[token]`
- `GET /api/market/properties` — Zillow market data
- `POST /api/investment/[watchlistId]` — ROI calculation
- `GET /api/reports/schedule-e` — Schedule E PDF
- `POST /api/billing/checkout` — Stripe checkout
- `POST /api/billing/portal` — Stripe customer portal
- `POST /api/webhooks/stripe` — Stripe webhook (no auth, HMAC verified)
- `POST /api/cron/notify` — Scheduled expiry reminders (cron, Bearer auth)
- `GET/POST /api/invite/[token]` — public, no auth
- `POST /api/portal/contracts/[id]/sign` — tenant signature submission

---

## Key Source Files

| File | What It Does |
|---|---|
| `web/lib/types.ts` | All TypeScript types and interfaces. Source of truth for domain models. |
| `web/lib/schemas.ts` | Zod validation schemas for all API request bodies |
| `web/lib/supabase-server.ts` | `createClient()` (RLS-aware SSR) and `createAdminClient()` (service role, bypasses RLS) |
| `web/lib/supabase.ts` | Client-side Supabase client |
| `web/lib/contract-context.ts` | `buildContext(contract)` — maps DB contract → DOCX template variables (Spanish) |
| `web/lib/notify.ts` | `sendResendEmail()`, `sendTwilioSms()`, `sendTenantInviteEmail()` |
| `web/lib/pdf-react.tsx` | React PDF renderer for contracts |
| `web/lib/pdf-template.ts` | DOCX → PDF pipeline |
| `web/lib/generate-docx.ts` | DOCX generation from `docxtemplater` |
| `web/lib/investment.ts` | Cap rate, cash flow, ROI calculation helpers |
| `web/lib/subscription.ts` | Plan limit checks (`PLAN_LIMITS`) |
| `web/lib/rate-limit.ts` | Upstash Redis rate limiter wrapper |
| `web/middleware.ts` | Auth routing, role-based redirects, locale cookie |
| `web/components/ContractBuilder.tsx` | Large form for contract create/edit (61KB) |
| `web/components/RenewalModal.tsx` | Contract renewal workflow (34KB) |

---

## Database Schema (Supabase Postgres)

### Core Tables
- **profiles** — `id, full_name, username, company_name, phone, email, role (landlord|tenant|manager), locale (es|en), plan (free|propietario|inversionista|enterprise)`
- **properties** — `id, owner_id, name, address, unit, city, state, zip, unit_count, bathroom_count, parking, latitude, longitude, jurisdiction`
- **tenants** — `id, owner_id, full_name, email, phone, ssn_last4, license_number, addresses (jsonb), employment (jsonb), emergency_contact (jsonb)`
- **contracts** — `id, owner_id, property_id, tenant_id, status (draft|sent|signed|expired|cancelled), type, lease_start, lease_end, rent_amount, late_fee_policy (jsonb), document URLs, snapshots (jsonb), parent_contract_id`

### Supporting Tables
- **contract_attachments** — supplementary files on a contract
- **contract_custom_sections** — per-contract custom clauses
- **contract_occupants** — co-tenants and guarantors
- **contract_templates** — user-uploaded DOCX templates
- **tenant_invites** — `token, contract_id, email, status (pending|accepted|expired)`
- **property_co_owners** — co-ownership relationships
- **property_manager** — delegated manager access
- **property_expenses** — tax deduction entries
- **notification_triggers** — `days_before_expiry, channels (sms|email), owner_id`
- **contract_notification_logs** — delivery tracking
- **subscriptions** — Stripe subscription state
- **watchlist** — Zillow property targets
- **investment_analyses** — ROI calculations
- **rea.zillow_unique** — market comparison data (separate schema)

### RLS Pattern
- Most tables: `owner_id = auth.uid()` policy
- Tenant portal: separate policies on `tenant_invites` and read-only contract access
- Service role (`createAdminClient()`) bypasses RLS — use for server-side writes that cross ownership boundaries

---

## Supabase Client Rules

```ts
// Server Components / API routes — respects RLS, uses session cookie
import { createClient } from '@/lib/supabase-server'
const supabase = createClient()

// Admin operations (cron, webhooks, cross-owner writes) — bypasses RLS
import { createAdminClient } from '@/lib/supabase-server'
const supabase = createAdminClient()

// Client Components
import { createClient } from '@/lib/supabase'
const supabase = createClient()
```

**Critical:** Any Server Component page that calls `createAdminClient()` **must** export:
```ts
export const dynamic = 'force-dynamic'
```
Otherwise Vercel caches the page and the admin client gets stale data.

---

## Auth & Role-Based Access

Roles are stored in `profiles.role`. Middleware enforces:
- `landlord` → can access `/dashboard/**`, blocked from `/portal`
- `tenant` → locked to `/portal/**`, redirected away from `/dashboard`
- `manager` → same access as landlord on delegated properties

Public routes (no auth required): `/`, `/pricing`, `/login`, `/signup`, `/forgot-password`, `/reset-password`, `/invite/**`, `/api/**`

---

## Subscription Plans

Defined in `web/lib/subscription.ts`:

| Plan | Key |
|---|---|
| Free | `free` |
| Propietario | `propietario` |
| Inversionista | `inversionista` |
| Enterprise | `enterprise` |

Check limits with `PLAN_LIMITS` constant before allowing feature access. Stripe price IDs are in env vars.

---

## Internationalization

- Framework: `next-intl`
- Locale cookie: `NEXT_LOCALE` (set in middleware from `profiles.locale`)
- Default: `es` (Spanish)
- Translation files: `web/messages/en.json`, `web/messages/es.json`
- Server config: `web/i18n/request.ts`

When adding new UI strings, add keys to **both** `en.json` and `es.json`.

---

## Document Generation

The core feature — contract generation flow:

1. `ContractBuilder.tsx` collects form data
2. `POST /api/generate` receives contract ID
3. Server calls `buildContext(contract)` → Spanish template variables
4. `docxtemplater` fills `templates/contract_template.docx`
5. Converted to PDF → stored in Supabase Storage
6. URL written back to `contracts` row

DOCX variables are all Spanish (see `lib/contract-context.ts` for full mapping).

---

## Environment Variables

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Email
RESEND_API_KEY=
FROM_EMAIL=

# SMS
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=

# Payments
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_PROPIETARIO=
STRIPE_PRICE_INVERSIONISTA=
STRIPE_CUSTOMER_PORTAL_URL=
STRIPE_2FA_SECRET=

# App
NEXT_PUBLIC_APP_URL=

# Cron (Bearer token for /api/cron/notify)
CRON_SECRET=
```

---

## Dev Workflow

```bash
cd web
npm run dev        # localhost:3000
npm run build      # production build check
npm run lint       # ESLint
npm run test       # Vitest
```

No local `node_modules` — run all commands from `web/`. The repo does not commit `node_modules`.

**Branching:** `main` ← `dev` ← feature branches. Always branch off `dev`. PR into `dev`, not `main`.

---

## Critical Patterns

1. **`force-dynamic` on admin pages** — any page using `createAdminClient()` must have `export const dynamic = 'force-dynamic'` at the top to prevent Vercel static caching.

2. **Zod validation on all API routes** — use schemas from `lib/schemas.ts`. Never trust raw `req.body`.

3. **Rate limiting** — apply `lib/rate-limit.ts` on all public-facing API routes (invite, sign, send).

4. **Stripe webhook** — verify HMAC signature before processing. Already handled in `api/webhooks/stripe/route.ts`.

5. **Cron route** — `api/cron/notify/route.ts` is called by Vercel Cron. Verifies `Authorization: Bearer ${CRON_SECRET}`.

6. **Tenant invite flow** — `tenant_invites` tokens are single-use. After redemption, status must be set to `accepted` before granting portal access.

7. **Contract snapshots** — when a contract is signed, property and tenant data are snapshotted into `contracts.property_snapshot` and `contracts.tenant_snapshot` (jsonb) so the signed document is immutable even if underlying records change.

---

## Supabase Migration Files

```
web/supabase/migrations/
  001_create_rea_schema.sql        — rea schema for Zillow market data
  002_investment_analyses.sql      — investment calculator tables
  003_add_desperation_score.sql    — property desperation metrics
  004_contract_templates.sql       — custom template storage
  005_property_counts_late_fees_snapshots.sql
  006_notification_system.sql      — email/SMS reminder triggers & logs
  007_property_expenses.sql        — expense tracking
  008_tenant_portal.sql            — tenant invite & portal access

supabase/migrations/
  20260523_pr_expansion.sql        — PR expansion (Ley 14, i18n, Act 60 features)
```

---
> Source: [TheF1Driver1/Contract_Portal](https://github.com/TheF1Driver1/Contract_Portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
