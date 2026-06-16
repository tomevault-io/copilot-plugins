## llm-agent-platform

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit Authorship

Every commit must end with exactly this trailer — no variations, no model name suffix:

```
Co-Authored-By: Claude <noreply@anthropic.com>
```

The primary author is always the git user (`valerii.bodnarchuk <bodnarchukvalole@gmail.com>`), set by the local git config. Claude is the co-author. This ensures the contribution graph on GitHub shows commits as belonging to the primary author.

## Commands

### NestJS Backend
```bash
npm run start:dev       # Dev server with hot reload
npm run build           # Compile TypeScript to dist/
npm run typecheck       # Type check without emit
npm test                # Run Jest tests (all suites, --runInBand)

# Targeted test runs
npx jest --testPathPatterns="ledger.service.spec" --verbose   # Ledger unit tests
npx jest --testPathPatterns="payment-lifecycle" --verbose     # Integration suite
npx jest --verbose                                            # All tests with detail
```

### Infrastructure
```bash
npm run docker:dev      # Start PostgreSQL + Redis only (for local dev)
npm run docker:up       # Full stack (app + postgres + redis + migrations)
npm run docker:down     # Stop full stack
```

### Database
```bash
npm run prisma:generate  # Regenerate Prisma client after schema changes
npm run prisma:migrate   # Run pending migrations
npm run prisma:seed      # Populate test data
```

### Fraud Engine (Python microservice)
```bash
cd fraud-engine
source venv/bin/activate
uvicorn app.main:app --reload --port 8000
pytest tests/
```

### Local dev setup
```bash
npm run docker:dev       # Start infra
cp .env.example .env    # Configure env
npm install && npm run prisma:generate
npm run prisma:migrate && npm run prisma:seed
npm run start:dev
```

Swagger docs: http://localhost:3000/api

## Architecture

This is a **NestJS/TypeScript** payment processing backend with a separate **Python/FastAPI** fraud engine microservice.

### Core Design Principles
- **Double-entry ledger**: All money movements record balanced DEBIT/CREDIT entries via `LedgerService`. Every ledger operation is wrapped in a Prisma `$transaction` for atomicity.
- **Idempotency**: Payments are deduplicated via Redis-cached `idempotencyKey` (24h TTL).
- **State machines**: Payouts (`PENDING → ELIGIBLE → PROCESSING → PAID/FAILED`) and Disputes (`OPEN → UNDER_REVIEW → WON/LOST/REFUNDED`) enforce transitions via `validateTransition()`.
- **Fail-open fraud**: If the fraud engine is unreachable, payouts default to `REVIEW` (not `BLOCK`).

### Ledger Account Types
`BUYER`, `SELLER`, `PLATFORM_FEE`, `ESCROW` — seller accounts have `allowNegative: true` to handle dispute losses.

### Payout Pipeline
1. `POST /payouts/create` → PENDING payout, ledger: ESCROW DEBIT → SELLER CREDIT (reserved)
2. `POST /payouts/:id/mark-eligible` → calls fraud engine → ELIGIBLE if ALLOW
3. `POST /payouts/:id/process` → Stripe Transfer, books ESCROW DEBIT + SELLER CREDIT + PLATFORM_FEE CREDIT → PAID
4. Retry: max 3 attempts, exponential backoff via `withRetry()`

### Fraud Engine Integration
- `FraudService` in `src/fraud/` calls the Python service at `FRAUD_ENGINE_URL` (default: `http://localhost:8000`)
- Scores 0.0–1.0: `< 0.3` → ALLOW, `0.3–0.7` → REVIEW, `≥ 0.7` → BLOCK
- Six rules: velocity, amount threshold, daily volume, failed history, new account, dispute rate

### Async Queue
BullMQ + Redis (`src/queue/`): `PayoutQueue` adds jobs, `PayoutProcessor` executes them, `PayoutScheduler` triggers daily payouts via cron. Bull Board admin UI at `/queue/jobs`.

### Reconciliation
Hourly (24h window) and daily (all-time) reconciliation syncs internal payout/ledger state with Stripe. Detects orphaned Stripe transfers and ledger imbalances.

### Ledger Integrity
`LedgerService.verifyIntegrity()` runs three `$queryRaw` aggregate checks: global debit/credit totals, per-transaction balance (`GROUP BY / HAVING`), and orphaned entry scan. Returns `LedgerIntegrityReport`. Called by `ReconciliationService.reconcileLedger()` and exposed at `GET /ledger/integrity`. Logs `warn` on imbalance, `info` on clean pass.

### Dispute Loss Allocation
1. Payout not yet released → refund from escrow, no seller loss
2. Payout released, seller has balance → reverse payout (seller absorbs loss)
3. Payout released, seller withdrew → seller balance goes negative → `payoutsBlocked = true` automatically

### Key Modules
| Module | Path | Purpose |
|--------|------|---------|
| Ledger | `src/ledger/` | Double-entry bookkeeping engine |
| Payment | `src/payment/` | Stripe PaymentIntent + escrow entry |
| Payout | `src/payout/` | Full payout lifecycle + retry logic |
| Fraud | `src/fraud/` | HTTP client to Python fraud engine |
| Dispute | `src/dispute/` | Chargeback handling + reversal |
| Seller | `src/seller/` | Stripe Connect KYC + account management |
| Webhook | `src/webhook/` | Stripe event processing |
| Queue | `src/queue/` | BullMQ async payout processing |
| Reconciliation | `src/reconciliation/` | Stripe/ledger sync |

### Environment Variables
See `.env.example`. Key vars: `DATABASE_URL`, `REDIS_HOST`/`REDIS_PORT` (or `REDIS_URL` in prod), `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `FRAUD_ENGINE_URL`.

### CI Pipeline (`.github/workflows/ci.yml`)
Runs: typecheck → Jest tests (with real PostgreSQL + Redis containers) → Docker build validation. Uses `sk_test_fake` / `whsec_fake` Stripe keys.

## Financial Invariants (Non-Negotiable)
- NEVER run destructive SQL (DROP, TRUNCATE, DELETE without WHERE)
- NEVER run migrations without explicit confirmation
- ALL financial state transitions must be inside Prisma $transaction
- Ledger entries are IMMUTABLE — never update, only append
- Idempotency required on all payment/payout mutations
- Correctness > simplicity for money-touching code

## Portfolio Context
- Target: senior fintech backend roles, DACH/UK, €100k+
- Code quality should reflect senior-level architectural decisions
- Every design decision should be defensible in a technical interview

---
> Source: [evanwillistech/llm-agent-platform](https://github.com/evanwillistech/llm-agent-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
