## jefitness

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start backend with hot-reload
npm run dev

# Start frontend with BrowserSync (separate terminal)
npm run dev:frontend

# Start both concurrently
npm run dev:full

# Run all tests
npm test

# Run only backend unit tests
npm run test:unit

# Run only frontend tests
npm run test:frontend

# Run tests with coverage
npm run test:coverage

# Run a single test file
npx jest src/tests/unit/dateUtils.test.js

# Lint and format
npm run lint
npm run lint:fix
npm run format

# Build Tailwind CSS
npm run build:css

# Sync Stripe plans to MongoDB StripePlan collection
npm run sync:plans

# Seed fitness programs into DB
npm run seed:programs

# Bust static asset cache (updates hash in cache-version.js)
npm run cache:bust
```

## Architecture Overview

**Stack**: Express 5 backend serving static HTML/JS frontend + MongoDB + Stripe payments.

The app is **not** a SPA — it's a multi-page HTML app where each page (`public/pages/*.html`) loads its own JS file from `public/js/`. The backend serves these files statically and maps clean URLs (e.g. `/dashboard` → `/pages/dashboard.html`) in `src/server.js`.

### Backend (`src/`)

- **Entry**: `src/server.js` — Express app, MongoDB connect, middleware stack, route mounting, cron jobs
- **Routes** (`src/routes/`) → **Controllers** (`src/controllers/`) pattern; all routes are prefixed `/api/v1/`
- **Auth**: JWT-based (`src/middleware/auth.js`). Tokens accepted from: httpOnly cookie (preferred), `Authorization: Bearer` header, or `x-auth-token` header. Role is always re-fetched from DB — never trust JWT claims for authorization
- **Stripe**: All Stripe logic goes through `src/services/stripe.js`. Subscription state is persisted to MongoDB (`Subscription` model) and synced from Stripe via webhooks (`src/routes/webhooks.js`) and on-demand via the `/refresh` endpoint
- **Date math**: All subscription date calculations (period end, days left, next renewal) must use `src/utils/dateUtils.js` — never raw `Date.now() + N * 86400000` arithmetic
- **Error classes**: `src/middleware/errorHandler.js` exports `AppError`, `ValidationError`, `AuthenticationError`, `AuthorizationError`, `NotFoundError`, `DatabaseError` — use these instead of generic `Error`
- **User cache**: `src/services/userCache.js` provides per-request and process-level caching for user lookups. Similar to frontend `auth-cache.js`, reduces DB round-trips. Usage: `const user = await userCache.findById(userId, req)` or `const users = await userCache.findByIds([id1, id2], req)`. Auto-initialized on every request via middleware

### Route Protection

Unprotected API routes: `/api/v1/products`, `/api/v1/subscriptions`, `/api/v1/programs`, `/api/v1/auth`

Protected routes use this middleware stack (in order): `auth` → `requireDataProcessingConsent` → (health routes also get `requireHealthDataConsent`) → `checkDataRestriction` → `apiLimiter` → `versioning`. Health-related routes (`/logs`, `/medical-documents`, `/workouts`, `/nutrition`) include the health consent check.

### Frontend (`public/js/`)

- `api.config.js` — resolves `API_BASE` (localhost vs production). Always use `window.ApiConfig.getAPI_BASE()` for API calls, never hardcode URLs. **Must load before `auth-cache.js`**
- `auth-cache.js` — singleton Promise cache for `/auth/me`. Fires one fetch per page load; all callers share the same Promise. Reset on failure so the next call retries
- `services/SubscriptionService.js` — thin fetch wrapper for all subscription API calls
- `subscriptions.js` — main subscription page logic: loads plans, renders active subscription summary, handles checkout flow
- `navbar-subscription.js` — loaded on every page to show subscription badge in the navbar
- `role-guard.js` — restricts page access by user role

### Subscription Flow

1. User selects plan → `subscriptions.js:selectPlan()` → POST `/api/v1/subscriptions/checkout` → Stripe Checkout redirect
2. On return: `?success=true&session_id=cs_xxx` → `verifyCheckoutSession` upserts the `Subscription` document from Stripe data
3. Stripe webhooks (`src/routes/webhooks.js`) keep the DB in sync for renewals, cancellations, etc. Webhook replay protection is handled via the `WebhookEvent` model with a 24-hour TTL
4. `getCurrentSubscription` has an auto-heal path: if `currentPeriodEnd` is missing or stale for an active sub, it re-fetches from Stripe inline

### Key Models

- `Subscription` — one document per Stripe subscription; `currentPeriodStart`/`currentPeriodEnd` are stored as JS `Date` objects (UTC). The pre-save hook auto-cancels on Stripe when `status` is set to `'canceled'`. Stripe is the authoritative source of truth — the cleanup job (`src/jobs.js`) always verifies with Stripe before marking a subscription canceled
- `StripePlan` — cached Stripe price/product data, populated by `npm run sync:plans`
- `User` — stores `stripeCustomerId` and `stripeSubscriptionId` as denormalized references; also embeds `MealLog[]` and `WorkoutLog[]` subdocuments directly on the user document
- `WebhookEvent` — persists processed Stripe event IDs with TTL for replay protection across restarts/instances

### Cron Jobs (`src/jobs.js`)

- **Subscription cleanup**: `0 0 * * *` (midnight) — verifies expired subs with Stripe before canceling
- **Renewal reminders**: `0 8 * * *` (8 AM) — emails users 3 and 7 days before renewal
- **Trainer daily schedule**: `0 0 * * *` (midnight) — sends daily appointment digest to trainers

### API Docs (Dev Only)

- Swagger UI: `http://localhost:PORT/api-docs`
- ReDoc: `http://localhost:PORT/redoc`

### Environment

Copy `.env.example` to `.env`. Required variables: `MONGO_URI`, `JWT_SECRET`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, and all `STRIPE_PRODUCT_*` / `STRIPE_PRICE_*` keys for the four subscription tiers (`1-month`, `3-month`, `6-month`, `12-month`). Optional: `USDA_API_KEY` (food search, falls back to `DEMO_KEY`), `CLEANUP_TIME` (minutes before deleting unverified accounts, default 30), `SLOW_REQUEST_THRESHOLD_MS`.

### Testing

- Jest config (`jest.config.js`) has two projects: `backend` (Node env, matches `src/tests/unit/**/*.test.js`) and `frontend` (jsdom, matches `public/tests/unit/**/*.test.js`)
- Backend test setup: `src/tests/unit/setup.js`; shared mocks: `src/tests/unit/mocks.js`
- Frontend test setup: `public/tests/setup-jsdom.js`
- Integration/E2E: Cypress (`cypress/`)
- Stress tests: `npm run test:stress`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jojo6550) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
