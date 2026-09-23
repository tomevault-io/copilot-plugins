## billing

> Billing & credits invariants


- Every credit mutation carries an `idempotencyKey`. Unique-violation on it
  means "already processed" — return the existing row silently.
- `syncStripeDataToDb(customerId)` in `src/lib/billing/sync.ts` is the ONLY
  code that writes subscription state. It fetches fresh from Stripe; event
  payloads are triggers, not sources of truth.
- The ledger is append-only: no UPDATE/DELETE on `credit_transactions`.
  Corrections are new compensating rows.
- Spends: single conditional UPDATE (`credit_balance >= amount` in WHERE)
  in the same transaction as the ledger insert. No check-then-write.
- Webhook: verify the Stripe signature on the RAW body (`request.text()`)
  before anything else. 200 fast; 500 on real failures so Stripe retries.
- Test subscription lifecycles with Stripe test clocks.

---
> Source: [nikandr-surkov/ai-saas-starter](https://github.com/nikandr-surkov/ai-saas-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
