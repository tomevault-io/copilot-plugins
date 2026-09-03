## booztfrog-com

> BooztFrog is a global SaaS platform for businesses to collect more customer reviews via NFC and QR products. It consists of two codebases sharing one PostgreSQL database:

# CLAUDE.md — BooztFrog Project

## What is this project?

BooztFrog is a global SaaS platform for businesses to collect more customer reviews via NFC and QR products. It consists of two codebases sharing one PostgreSQL database:

1. **booztfrog-frontend** — Next.js 16 app (storefront + dashboard) hosted on Vercel
2. **booztfrog-api** — Laravel 12 API backend hosted on VPS

Full spec: See `BOOZTFROG-SPEC.md` in project root.

---

## Architecture Rules

- **Frontend and backend are separate repositories/projects.** They communicate exclusively via REST API.
- **Laravel is API-only.** No Blade views except for `go.booztfrog.com` redirect landing pages (for performance).
- **All business data is scoped by user_id/business_id.** Never return data belonging to other users.
- **UUIDs for all primary keys.** No auto-incrementing integers.
- **All money values stored in cents** (integer). Never use floats for currency.
- **Translatable content uses JSONB columns:** `{"en": "Hello", "nb": "Hei"}`. Do NOT create separate translation tables for product content.

---

## Tech Stack

### Frontend
- **Framework:** Next.js 16 with App Router (TypeScript strict mode)
- **Styling:** Tailwind CSS
- **Components:** shadcn/ui (copied into project, not imported as dependency)
- **i18n:** next-intl with path-based routing (`/en/...`, `/nb/...`)
- **State:** React Query (TanStack Query) for server state, Zustand for client state if needed
- **Forms:** React Hook Form + Zod validation
- **Charts:** Recharts
- **HTTP:** Axios or fetch wrapper in `lib/api.ts`
- **Auth:** Sanctum token stored in httpOnly cookie or secure localStorage

### Backend
- **Framework:** Laravel 12 (PHP 8.5)
- **Auth:** Laravel Sanctum (token-based)
- **Billing:** Laravel Cashier (Stripe)
- **Database:** PostgreSQL 16
- **Cache/Queue:** Redis
- **Queue driver:** Redis (not database)
- **File storage:** S3-compatible (Cloudflare R2 or AWS S3)
- **GeoIP:** MaxMind GeoLite2 (local database, no API calls)
- **Email:** Postmark or Resend via Laravel Mail

---

## Coding Standards

See **`coding-standards.md`** for the complete coding standards document covering TypeScript, PHP/Laravel, database, API responses, i18n, security, performance, and testing.

Key points:
- Strict TypeScript — no `any` types
- Server components by default — `'use client'` only when needed
- All API calls through `lib/api.ts` — never call fetch directly in components
- All user-facing strings via `useTranslations()` — zero hardcoded text
- Laravel: Form Requests for validation, API Resources for responses, Policies for auth
- Services for business logic — controllers stay thin
- Jobs for async work — never block the request
- UUIDs for all primary keys, money in cents (integers)

---

## File Structure Conventions

### Frontend

```
app/[locale]/(storefront)/   → Public marketing pages
app/[locale]/(auth)/          → Login, register, forgot password
app/[locale]/(dashboard)/     → Authenticated SaaS dashboard
components/storefront/        → Storefront-specific components
components/dashboard/         → Dashboard-specific components
components/shared/            → Shared across both
components/ui/                → shadcn/ui base components
lib/                          → Utilities, API client, helpers
hooks/                        → Custom React hooks
messages/                     → i18n translation files (JSON)
```

### Backend

```
app/Http/Controllers/Auth/        → Authentication
app/Http/Controllers/Storefront/  → Public API (products, checkout)
app/Http/Controllers/Dashboard/   → Authenticated API (business, devices, analytics)
app/Http/Controllers/Admin/       → Admin-only API
app/Http/Controllers/Redirect/    → go.booztfrog.com handler
app/Http/Requests/                → Form validation
app/Http/Resources/               → JSON response transformation
app/Models/                       → Eloquent models
app/Services/                     → Business logic
app/Jobs/                         → Queued async jobs
app/Events/                       → Event classes
app/Policies/                     → Authorization policies
routes/api.php                    → API routes
routes/redirect.php               → NFC redirect routes
```

---

## Important Implementation Notes

### NFC Redirect (`go.booztfrog.com/{code}`)
- This is the MOST performance-critical route. Target < 100ms response time.
- Cache device → business → platforms mapping in Redis
- Log scans asynchronously via queue job (never block the redirect)
- Use a minimal middleware stack (just rate limiting)
- For multi-platform selection: render a simple Blade template, NOT a Next.js page
- GeoIP lookup happens in the queue job, not in the redirect handler

### Stripe Checkout Flow
- Use Stripe Checkout Sessions for one-time product purchases
- Use Laravel Cashier for subscription management
- Always verify webhook signatures
- Handle all critical webhook events: `checkout.session.completed`, `customer.subscription.*`, `invoice.payment_failed`

### i18n
- Default language: English (`en`)
- URL structure: `booztfrog.com/{locale}/path` (e.g., `/nb/products`)
- API accepts `Accept-Language` header to return translated content
- All JSONB translation columns fall back to `en` if requested locale is missing
- NFC landing pages detect browser language automatically

### Analytics
- Raw scans go to `scans` table
- Daily aggregation job runs at midnight UTC → populates `scan_aggregates`
- Dashboard queries read from `scan_aggregates` for performance
- Real-time "today" data queries `scans` table directly (small dataset for current day)

---

## Testing Strategy

- **Backend:** PHPUnit — Feature tests for all API endpoints, Unit tests for services
- **Frontend:** Vitest for unit tests, Playwright for E2E
- **Minimum coverage:** All API endpoints must have feature tests
- **Test database:** Use SQLite for fast tests, PostgreSQL for CI

---

## Deployment

- **Frontend:** Push to `main` branch → Vercel auto-deploys
- **Backend:** Push to `main` branch → Laravel Forge auto-deploys to VPS
- **Database migrations:** Run automatically on deploy via Forge deployment script
- **Queue workers:** Managed by Supervisor (configured via Forge)

---

## Common Commands

### Frontend
```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript check
```

### Backend
```bash
php artisan serve                    # Start dev server
php artisan migrate                  # Run migrations
php artisan db:seed                  # Seed database
php artisan queue:work               # Start queue worker
php artisan schedule:run             # Run scheduled tasks
php artisan test                     # Run tests
php artisan make:model Name -mfcr    # Model + migration + factory + controller + resource
```

---
> Source: [DonCrypto78/booztfrog.com](https://github.com/DonCrypto78/booztfrog.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
