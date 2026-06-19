## cstack

> >


# CodersHub Innovations — Company-as-a-Repo SKILL

> **One clone. One command. Your entire company runs.**

---

## 0. Quick-Start (for any contributor)

```bash
git clone https://github.com/codershublabs/company-os.git
cd company-os
cp .env.example .env.local        # fill in secrets (see §8)
pnpm install
pnpm db:migrate                   # applies all Supabase migrations
pnpm dev                          # boots all apps in parallel via Turbo
```

Open `http://localhost:3000` → main site  
Open `http://localhost:3001` → member portal  
Open `http://localhost:3002` → admin / ops dashboard  

---

## 1. Company DNA

| Field | Value |
|---|---|
| **Legal name** | CodersHub Innovations |
| **Brand name** | CodersHub Labs |
| **HQ** | Yaoundé, Cameroon 🇨🇲 |
| **Mission** | Empower African builders through hands-on engineering, structured incubation, and community collaboration |
| **Target market** | Africa-first → Global |
| **Languages** | Bilingual: English + French (i18n via `next-intl`) |
| **Stack** | TypeScript · Next.js 14 App Router · Supabase · Vercel · Turborepo · pnpm |
| **Payments** | MTN MoMo · Orange Money · Stripe · PayPal |
| **Integrations** | GitHub · Slack/Discord · WhatsApp (via Twilio/WhatsApp Cloud API) |

---

## 2. Ecosystem Arms

Each arm is a **Turborepo app or package**. They share a single Supabase instance but have isolated schemas/modules.

| Arm | Slug | Purpose | Port (dev) |
|---|---|---|---|
| Main Website | `web` | Marketing, landing pages, blog, SEO | 3000 |
| Member Portal | `portal` | Student/member dashboard | 3001 |
| Admin / OpsDesk | `admin` | Internal ops, tasks, docs, automation | 3002 |
| DevCore | `devcore` | Bootcamp & course management | 3003 |
| CodersHub Labs | `labs` | Startup incubation dashboard | 3004 |
| HubConnect | `hubconnect` | Community, networking, events | 3005 |
| DesignHub | `designhub` | Design services, portfolio, briefs | 3006 |
| TechSpire | `techspire` | Research arm, publications, reports | 3007 |
| OpsDesk | `opsdesk` | Internal task tracker, SOPs, HR | 3002 (shared with admin) |

---

## 3. Monorepo Folder Structure

```
company-os/
├── apps/
│   ├── web/              # Main marketing site (Next.js)
│   ├── portal/           # Member portal (Next.js)
│   ├── admin/            # Admin + OpsDesk (Next.js)
│   ├── devcore/          # Bootcamp management (Next.js)
│   ├── labs/             # Incubation dashboard (Next.js)
│   ├── hubconnect/       # Community platform (Next.js)
│   ├── designhub/        # Design arm (Next.js)
│   ├── techspire/        # Research arm (Next.js)
│   └── api/              # Shared API layer / edge functions (Hono.js)
│
├── packages/
│   ├── ui/               # Shared component library (Shadcn/ui base)
│   ├── db/               # Supabase client, types, migrations
│   ├── auth/             # Auth helpers (Supabase Auth)
│   ├── payments/         # MoMo + Stripe + PayPal abstraction layer
│   ├── i18n/             # English + French translations
│   ├── notifications/    # Email (Resend) + WhatsApp + Slack/Discord
│   ├── config/           # Shared ESLint, TS, Tailwind configs
│   └── utils/            # Shared utilities
│
├── supabase/
│   ├── migrations/       # All DB migrations (versioned)
│   ├── seed.sql          # Dev seed data
│   └── functions/        # Supabase Edge Functions
│
├── .github/
│   ├── workflows/        # CI/CD pipelines
│   └── CODEOWNERS        # Team ownership map
│
├── docs/
│   ├── architecture.md
│   ├── runbooks/         # Operational runbooks per arm
│   └── onboarding.md     # New team member guide
│
├── scripts/
│   ├── setup.sh          # First-time setup script
│   ├── seed-dev.ts       # Populate dev environment
│   └── deploy.sh         # Manual deploy trigger
│
├── turbo.json
├── pnpm-workspace.yaml
├── .env.example
└── package.json
```

---

## 4. Database Schema (Supabase / PostgreSQL)

All tables live in Supabase. Row Level Security (RLS) is **always on**.

### Core Tables

```sql
-- Identity
profiles           (id, user_id, full_name, avatar_url, role, arm, lang, created_at)
roles              (id, name, permissions[])   -- e.g. admin, mentor, student, founder

-- DevCore (Bootcamps)
cohorts            (id, name, arm, start_date, end_date, status, capacity)
enrollments        (id, user_id, cohort_id, status, payment_id, created_at)
modules            (id, cohort_id, title, order, content_url)
assignments        (id, module_id, title, due_date, max_score)
submissions        (id, assignment_id, user_id, content_url, score, feedback)

-- CodersHub Labs (Incubation)
startups           (id, founder_id, name, stage, sector, deck_url, status)
incubation_apps    (id, user_id, startup_id, status, reviewer_id, notes)
milestones         (id, startup_id, title, due_date, completed_at)

-- HubConnect (Community)
posts              (id, author_id, content, type, arm, likes, created_at)
events             (id, title, arm, date, location, rsvp_count, stream_url)
rsvps              (id, event_id, user_id, status)
mentorship_requests(id, mentee_id, mentor_id, status, topic, scheduled_at)

-- OpsDesk (Internal)
tasks              (id, assignee_id, title, status, priority, due_date, arm)
documents          (id, author_id, title, content, type, arm, version)
team_members       (id, user_id, arm, title, joined_at, status)

-- Payments
payments           (id, user_id, amount, currency, gateway, gateway_ref, status, metadata)
subscriptions      (id, user_id, plan, status, current_period_end, gateway)
plans              (id, name, price_xaf, price_usd, interval, features[])

-- CRM / Leads
leads              (id, name, email, phone, source, status, arm, notes, created_at)
communications     (id, lead_id, user_id, channel, content, sent_at)

-- TechSpire (Research)
publications       (id, author_id, title, abstract, pdf_url, published_at, tags[])
```

---

## 5. Authentication & Roles

Use **Supabase Auth** with email/password + magic link.

### Role Hierarchy

```
super_admin       → full access to all arms
arm_admin         → full access to their arm only
mentor            → read/write in DevCore + HubConnect
founder           → access to Labs incubation dashboard
student/member    → portal access (enrollments, community, jobs)
guest             → public pages only
```

### Auth Flow

1. Sign up → profile created via `profiles` trigger
2. Role assigned by `super_admin` or auto-assigned on enrollment
3. JWT custom claims include `role` and `arm`
4. RLS policies check `auth.jwt() ->> 'role'`

---

## 6. Payment Architecture (`packages/payments`)

Single abstraction layer over all gateways:

```typescript
// packages/payments/index.ts
interface PaymentProvider {
  createPayment(opts: PaymentOptions): Promise<PaymentResult>
  verifyPayment(ref: string): Promise<PaymentStatus>
  createSubscription(opts: SubscriptionOptions): Promise<SubscriptionResult>
}

// Providers
class MoMoProvider implements PaymentProvider  // MTN + Orange via CinetPay or NotchPay
class StripeProvider implements PaymentProvider
class PayPalProvider implements PaymentProvider

// Currency handling
// XAF (CFA Franc) is the base currency for Cameroon
// Stripe/PayPal receive USD/EUR with live conversion
```

### Supported Plans

| Plan | XAF/month | USD/month | Access |
|---|---|---|---|
| Free | 0 | 0 | Community, public events |
| Member | 5,000 | ~8 | Portal, HubConnect, job board |
| Builder | 15,000 | ~25 | DevCore courses, DesignHub |
| Founder | 30,000 | ~50 | Labs incubation + all above |
| Enterprise | Custom | Custom | White-label, API access |

---

## 7. Notifications (`packages/notifications`)

```typescript
// Channels available
- Email:    Resend (transactional) + Mailchimp (marketing)
- WhatsApp: WhatsApp Cloud API via Meta (primary mobile channel for Cameroon)
- Slack:    Internal team notifications
- Discord:  Community notifications
- In-app:   Supabase Realtime + push via web push API
```

### Trigger Map

| Event | Email | WhatsApp | Slack | Discord |
|---|---|---|---|---|
| New enrollment | ✅ | ✅ | ✅ (arm channel) | - |
| Assignment due | ✅ | ✅ | - | - |
| Payment received | ✅ | ✅ | ✅ | - |
| New community post | - | - | - | ✅ |
| Incubation update | ✅ | ✅ | ✅ | - |
| New lead (CRM) | - | - | ✅ | - |
| Event reminder | ✅ | ✅ | - | ✅ |

---

## 8. Environment Variables (`.env.example`)

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Payments
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=
PAYPAL_CLIENT_ID=
PAYPAL_CLIENT_SECRET=
CINETPAY_API_KEY=           # MTN MoMo + Orange Money (CinetPay aggregator)
CINETPAY_SITE_ID=
NOTCHPAY_PUBLIC_KEY=        # Alternative MoMo provider

# Notifications
RESEND_API_KEY=
WHATSAPP_TOKEN=             # Meta WhatsApp Cloud API
WHATSAPP_PHONE_NUMBER_ID=
SLACK_BOT_TOKEN=
SLACK_SIGNING_SECRET=
DISCORD_BOT_TOKEN=
DISCORD_GUILD_ID=

# GitHub
GITHUB_TOKEN=               # For GitHub integration in OpsDesk
GITHUB_ORG=codershublabs

# App
NEXT_PUBLIC_APP_URL=https://codershublabs.com
NEXT_PUBLIC_PORTAL_URL=https://portal.codershublabs.com
ADMIN_SECRET_KEY=           # Internal admin API calls

# i18n
NEXT_PUBLIC_DEFAULT_LOCALE=en
```

---

## 9. i18n (Bilingual: English + French)

```
packages/i18n/
├── en/
│   ├── common.json
│   ├── devcore.json
│   ├── labs.json
│   ├── hubconnect.json
│   └── ...
└── fr/
    ├── common.json
    ├── devcore.json
    ├── labs.json
    ├── hubconnect.json
    └── ...
```

- Uses `next-intl` in all apps
- Locale detected from browser → stored in user profile
- URL structure: `/en/...` and `/fr/...`
- Admin panel always bilingual; emails sent in user's preferred language

---

## 10. CI/CD (GitHub Actions → Vercel)

```yaml
# .github/workflows/ci.yml
on: [push, pull_request]
jobs:
  lint-typecheck:   pnpm lint && pnpm typecheck
  test:             pnpm test (Vitest)
  build:            pnpm build (Turbo)

# .github/workflows/deploy.yml
on:
  push:
    branches: [main]
jobs:
  deploy-web:       vercel deploy --prod (apps/web)
  deploy-portal:    vercel deploy --prod (apps/portal)
  deploy-admin:     vercel deploy --prod (apps/admin)
  # ... one deploy job per app
```

Each app maps to its own Vercel project:
- `web` → `codershublabs.com`
- `portal` → `portal.codershublabs.com`
- `admin` → `admin.codershublabs.com`
- `devcore` → `devcore.codershublabs.com`
- etc.

---

## 11. Team & Access Map

| Role | Arm | Repo Access | Admin Access |
|---|---|---|---|
| Founder / CTO | All | Owner | Super Admin |
| DevCore Lead | DevCore | Write | Arm Admin |
| Labs Lead | Labs | Write | Arm Admin |
| HubConnect Lead | HubConnect | Write | Arm Admin |
| DesignHub Lead | DesignHub | Write | Arm Admin |
| TechSpire Lead | TechSpire | Write | Arm Admin |
| OpsDesk Manager | OpsDesk | Write | Arm Admin |
| Mentors | DevCore, HubConnect | Read | Mentor role |
| Interns | Assigned arm | Read | Student role |

---

## 12. Feature Development Workflow

When adding any feature to this repo:

1. **Check this SKILL.md first** — understand which arm/module is affected
2. **Identify the app** — `apps/<arm>/` or `packages/<shared>/`
3. **DB change?** → Create a new migration in `supabase/migrations/`
4. **New API?** → Add route in `apps/api/src/routes/<arm>/`
5. **New UI?** → Build in `packages/ui/` if shared, or `apps/<arm>/components/` if arm-specific
6. **New notification?** → Add trigger in `packages/notifications/triggers/`
7. **New payment plan?** → Update `packages/payments/plans.ts` and `plans` table
8. **i18n string?** → Add to BOTH `en/` and `fr/` locale files
9. **Tests** → `apps/<arm>/__tests__/` using Vitest
10. **PR** → Must pass CI + reviewed by arm lead

---

## 13. Arm-Specific Runbooks

Read the relevant file in `docs/runbooks/` before working on an arm:

| File | Covers |
|---|---|
| `runbooks/devcore.md` | Cohort setup, assignment grading, certificate generation |
| `runbooks/labs.md` | Incubation pipeline, startup evaluation rubric |
| `runbooks/hubconnect.md` | Event creation, moderation, mentorship matching |
| `runbooks/designhub.md` | Design brief intake, delivery workflow |
| `runbooks/techspire.md` | Publication submission, peer review process |
| `runbooks/opsdesk.md` | Task management, SOP templates, HR flows |
| `runbooks/crm.md` | Lead intake, follow-up sequences, conversion tracking |
| `runbooks/payments.md` | Gateway configs, refund policy, XAF/USD handling |

---

## 14. Africa-First Design Principles

These are NON-NEGOTIABLE constraints that apply to every feature:

1. **Low bandwidth first** — All pages must score ≥90 on Lighthouse Performance. Lazy load images. No autoplay video.
2. **Mobile first** — 80%+ of Cameroon users are on mobile. Every UI starts from 320px.
3. **MoMo before cards** — MTN MoMo / Orange Money must always be the first/default payment option shown.
4. **WhatsApp over email** — For critical user-facing notifications, WhatsApp is primary. Email is secondary.
5. **Offline resilience** — Cache critical pages with service workers. Show graceful degraded states.
6. **French parity** — No feature ships in English-only. Both locales must be complete before merge.
7. **XAF pricing first** — All prices displayed in XAF by default; USD/EUR shown secondary.
8. **Data cost awareness** — No feature that auto-downloads large files without user consent.

---

## 15. Key Commands Reference

```bash
# Development
pnpm dev                    # Boot all apps
pnpm dev --filter=portal    # Boot only the portal
pnpm dev --filter=devcore   # Boot only DevCore

# Database
pnpm db:migrate             # Run all pending migrations
pnpm db:reset               # Reset DB + re-seed (dev only)
pnpm db:types               # Regenerate TypeScript types from Supabase

# Code quality
pnpm lint                   # ESLint all packages
pnpm typecheck              # tsc --noEmit all packages
pnpm test                   # Vitest all packages
pnpm format                 # Prettier all packages

# Build & Deploy
pnpm build                  # Build all apps (Turbo)
pnpm deploy:all             # Deploy all apps to Vercel
pnpm deploy --filter=web    # Deploy only the main site

# Utilities
pnpm generate:migration     # Scaffold a new DB migration file
pnpm seed:dev               # Seed dev DB with sample data
pnpm i18n:check             # Verify all translation keys exist in both locales
```

---

## 16. GitHub Repo Setup Checklist

Before the first push, ensure:

- [ ] Repo created: `codershublabs/company-os` (private)
- [ ] Branch protection on `main` (require PR + CI pass)
- [ ] Vercel projects created for each app (8 projects)
- [ ] Supabase project created, `.env` populated
- [ ] `CODEOWNERS` file mapping arms to leads
- [ ] GitHub Secrets populated (all env vars from §8)
- [ ] Slack webhook connected to `#deployments` channel
- [ ] WhatsApp Cloud API app approved in Meta Developer Portal
- [ ] CinetPay/NotchPay account created for MoMo
- [ ] Stripe account created (use Cameroon-registered business)
- [ ] Resend domain verified for `@codershublabs.com`

---

*This SKILL.md is the source of truth for the CodersHub company-os repo. Update it whenever a major architectural decision changes. All contributors must read §4, §8, and §14 before their first commit.*

---
> Source: [Ebesoh-Adrian/cstack](https://github.com/Ebesoh-Adrian/cstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
