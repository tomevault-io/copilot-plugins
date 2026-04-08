## dfwsc2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Payment portal for DFW Software Consulting clients integrating Stripe Connect. Clients onboard via Stripe Express accounts and process payments through the platform.

## Monorepo Structure
- `backend/` — Fastify/TypeScript API server
- `front/` — React frontend
- Root `package.json` delegates to workspace subdirectories

## Commands

### Dev stack (Docker — preferred for all backend work)
```sh
make up           # Start dev stack (api, web, db, mailhog, stripe-cli)
make up-build     # Build + start
make down         # Stop
make down-v       # Stop + remove volumes (destroys pgdata)
make logs         # Tail API logs
make sh           # Shell into API container
make test         # Run backend tests inside running container
make test-up      # Start stack then run backend tests
```

### Running tests
```sh
# Backend (must be inside container or with DATABASE_URL set):
make test
# Single test file:
docker compose -f docker-compose.base.yml -f docker-compose.dev.yml exec api npx vitest run src/__tests__/app.test.ts

# Frontend:
cd front && npm test        # or npx vitest
```

### Database migrations
```sh
npm run db:generate   # Generate migration from schema changes
npm run db:migrate    # Apply migrations
```

### Swagger UI
Available at `http://localhost:4242/docs` in dev. Disabled when `ENABLE_SWAGGER=false` or `NODE_ENV=production`.

## Architecture

### Request auth — two separate schemes
- **Client routes** (`POST /payments/create`): `X-Api-Key` header. Auth uses a two-field strategy — `apiKeyLookup` (SHA256 hash for O(1) DB lookup) + `apiKeyHash` (bcrypt for verification). A legacy fallback iterates all clients with null `apiKeyLookup` for backward compatibility.
- **Admin routes**: JWT Bearer token obtained from `POST /api/v1/auth/login`. Token must carry `role: "admin"` claim.

### Payment flow — USE_CHECKOUT toggle
`USE_CHECKOUT` env var switches between two Stripe payment modes:
- `false` (default): Creates a **PaymentIntent** via Stripe Connect. Returns `clientSecret` for frontend Elements.
- `true`: Creates a **Checkout Session**. Returns a `url` for redirect. Requires `lineItems` in request body.

Both modes use `application_fee_amount` from `DEFAULT_PROCESS_FEE_CENTS` and require an `Idempotency-Key` header.

### Onboarding flow
1. Admin calls `POST /api/v1/accounts` or `POST /api/v1/onboard-client/initiate` → creates client record + `onboardingTokens` row (status: `pending`) + returns API key (only time it's shown plaintext).
2. Client visits frontend `/onboard?token=...` → frontend calls `GET /api/v1/onboard-client?token=...` → creates Stripe Express account + account link, transitions token to `in_progress`, stores CSRF `state` with 30-min expiry.
3. Stripe redirects to `GET /api/v1/connect/callback?client_id=...&state=...&account=...` → validates state, links `stripeAccountId` to client, marks token `completed` in a DB transaction.
4. `GET /api/v1/connect/refresh?token=...` regenerates an account link for incomplete onboarding sessions.

### Rate limiting
In-memory sliding-window rate limiter (`lib/rate-limit.ts`). Not Redis-backed — not suitable for multi-instance deployments. Admin/onboard routes: 10 req/min per IP. Payment routes: 20 req/min per Stripe account ID (falls back to IP).

### Database schema (Drizzle ORM, PostgreSQL 17)
Six tables:
- `clients` — `id`, `name`, `email`, `apiKeyHash`, `apiKeyLookup`, `stripeAccountId`, `status` (active/inactive), `groupId` (FK→client_groups, nullable), `paymentSuccessUrl`, `paymentCancelUrl`, `processingFeePercent`, `processingFeeCents`
- `client_groups` — `id`, `name`, `status` (active/inactive), `processingFeePercent`, `processingFeeCents`, `paymentSuccessUrl`, `paymentCancelUrl`
- `onboarding_tokens` — `clientId` (FK→clients), `token`, `email`, `status`, `state`, `stateExpiresAt`
- `webhook_events` — idempotency table for Stripe webhook deduplication
- `subscriptions` — `id`, `clientId` (FK→clients), `amountCents`, `description`, `interval` (monthly/quarterly/yearly), `totalPayments`, `paymentsMade`, `status` (active/paused/cancelled), `nextBillingDate`
- `invoices` — `id`, `clientId` (FK→clients), `subscriptionId` (nullable FK→subscriptions), `amountCents`, `description`, `dueDate`, `status` (pending/paid/cancelled), `paymentToken` (unique), `paidAt`

### Environment variables
Required: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `FRONTEND_ORIGIN`, `USE_CHECKOUT`, `DATABASE_URL`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `JWT_SECRET`.

Optional: `PORT` (default 4242), `API_BASE_URL`, `DEFAULT_PROCESS_FEE_CENTS`, `SMTP_FROM`, `ENABLE_SWAGGER`, `JWT_EXPIRY` (default `1h`), `SETUP_FLAG_PATH`.

Bootstrap mode (skip admin creds on first run): set `ALLOW_ADMIN_SETUP=true` + `ADMIN_SETUP_TOKEN`.

### Full API route map
All routes prefixed `/api/v1` except `/docs`.

| Method | Path | Auth |
|--------|------|------|
| GET | `/api/v1/health` | none |
| POST | `/api/v1/auth/login` | none |
| GET | `/api/v1/auth/setup/status` | none (bootstrap check) |
| POST | `/api/v1/auth/setup` | optional token (bootstrap only) |
| POST | `/api/v1/accounts` | admin JWT + rate limit |
| POST | `/api/v1/onboard-client/initiate` | admin JWT + rate limit |
| POST | `/api/v1/onboard-client/resend` | admin JWT + rate limit |
| GET | `/api/v1/onboard-client` | rate limit |
| GET | `/api/v1/connect/refresh` | rate limit |
| GET | `/api/v1/connect/callback` | none (state CSRF) |
| POST | `/api/v1/payments/create` | API key + rate limit |
| GET | `/api/v1/reports/payments` | admin JWT |
| GET | `/api/v1/clients` | admin JWT |
| PATCH | `/api/v1/clients/:id` | admin JWT |
| GET | `/api/v1/groups` | admin JWT |
| POST | `/api/v1/groups` | admin JWT |
| PATCH | `/api/v1/groups/:id` | admin JWT |
| GET | `/api/v1/invoices` | admin JWT |
| POST | `/api/v1/invoices` | admin JWT |
| PATCH | `/api/v1/invoices/:id` | admin JWT |
| GET | `/api/v1/invoices/pay/:token` | none (public) |
| POST | `/api/v1/invoices/pay/:token` | none (public mock payment) |
| GET | `/api/v1/subscriptions` | admin JWT |
| POST | `/api/v1/subscriptions` | admin JWT |
| GET | `/api/v1/subscriptions/:id` | admin JWT |
| PATCH | `/api/v1/subscriptions/:id` | admin JWT |
| POST | `/api/v1/webhooks/stripe` | Stripe signature |

### Fee calculation priority
When a payment is created, the platform fee is resolved via this 5-level chain (first match wins):
1. **Client `processingFeePercent`** — percentage of payment amount
2. **Client `processingFeeCents`** — flat fee in cents
3. **Group `processingFeePercent`** — if client belongs to a group
4. **Group `processingFeeCents`** — if client belongs to a group
5. **`DEFAULT_PROCESS_FEE_CENTS`** env var — global default (defaults to `0`)

## Git Hooks & Commit Standards

### Husky hooks
- **pre-commit**: 14 guards — blocks commits to `main`, `.env` files, conflict markers, files >500KB, missing lockfile, secrets (`secretlint`), `console.log`/`debugger`, `.only`/`.skip` in tests, schema changes without migrations, out-of-sync doc files. Runs `lint-staged` (Biome auto-fix) + TypeScript type-check + frontend tests.
- **commit-msg**: enforces conventional commits via `commitlint` (`commitlint.config.js`)
- **pre-push**: blocks force-push to `main`; runs backend tests inside Docker (requires `make up`)

### Commit message format (conventional commits)
```
type(scope): subject
```
Allowed types: `feat` `fix` `docs` `style` `refactor` `chore` `test` `perf` `ci` `build` `revert`

Rules: lowercase subject, no trailing period, 100-char header limit.

### Escape hatches
| Situation | Flag |
|-----------|------|
| Skip migration check | `SKIP_MIGRATION_CHECK=1 git commit` |
| Allow console.log | `ALLOW_CONSOLE=1 git commit` |
| Skip backend tests on push | `SKIP_DOCKER_TESTS=1 git push` |

## Docker Environment
- `api`: port 4242
- `web`: port 8080
- `db`: PostgreSQL, port 5432 (localhost only in prod)
- `mailhog`: SMTP testing, UI port 8025
- `stripe-cli`: webhook forwarding in dev

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DFW-Software-Consulting)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DFW-Software-Consulting)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
