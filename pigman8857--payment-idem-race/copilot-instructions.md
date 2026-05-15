## payment-idem-race

> Learning project for **idempotency** and **race conditions** in a payment + stock flow. NestJS + MongoDB. Code evolves through phases — each phase reproduces a real bug first, then introduces the fix.

# CLAUDE.md — payment-idem-race

## Purpose

Learning project for **idempotency** and **race conditions** in a payment + stock flow. NestJS + MongoDB. Code evolves through phases — each phase reproduces a real bug first, then introduces the fix.

See `PLAN.md` for the full phased plan. See `docs/` for per-phase results and analysis.

## Tech stack

- **NestJS** (TypeScript)
- **MongoDB 7** — used for everything: primary store, cache, locks, outbox. **No Redis.**
- **Mongoose** ODM
- **autocannon** — load testing (npm dep, no install needed)
- **k6** — load testing for multi-step scenarios (separate binary, optional)
- Single-node **replica set** in Docker Compose — required for transactions and change streams

## How phases work

The checkout logic in `src/checkout/` is rewritten each phase. Other modules (products, orders, payments) evolve incrementally. New collections are introduced as needed.

| Phase | Status | Core learning | Key change |
|---|---|---|---|
| 0 | **done** | See the bugs | Naive read-modify-write, no idempotency |
| 1 | **designing** | Idempotency keys | `idempotency_keys` collection, unique index, webhook dedup |
| 2 | not started | Race condition fixes | Atomic `findOneAndUpdate`, optimistic locking, transactions |
| 3 | not started | Reserve / confirm | `stock_reservations` with TTL index |
| 4 | not started | Outbox + change streams | Reliable side-effects after payment |
| 5 | not started | Distributed locks | `locks` collection (only if needed) |

Phase 0 code is marked with `// Phase 0:` comments at the places that will be replaced. Do not clean these up — they are intentional signposts.

Each phase has two docs in `docs/`:
- **Design doc** — written before coding. Captures concepts, decisions, edge cases, HTTP codes, data model. Named `phase-N.md`.
- **Results doc** — written after the load test. Captures actual test output, diagrams, bug analysis. Also named `phase-N.md` (same file, results section appended at the bottom).

## Key files

| File | Role |
|---|---|
| `src/checkout/checkout.service.ts` | The main flow — rewritten each phase |
| `src/checkout/checkout.controller.ts` | HTTP entry point — idempotency header added in Phase 1 |
| `src/products/products.service.ts` | Stock operations — `naiveDecrementStock` replaced in Phase 2 |
| `src/payments/payments.service.ts` | Fake payment provider — 100–300ms delay is intentional (widens race window) |
| `src/idempotency/` | Phase 1 — interceptor + schema + service for idempotency key handling (not yet created) |
| `src/seed.ts` | Seeds one product with `stock: 10` |
| `loadtests/` | One autocannon + one k6 script per phase |
| `docs/phase-0.md` | Phase 0 design + load test results + bug analysis |
| `docs/phase-1.md` | Phase 1 design doc — idempotency concepts, unique index, edge cases, HTTP codes |
| `docs/phase-2.md` | Phase 2 design doc — three race condition approaches, decision framework, when to use which |
| `docs/external-api-patterns.md` | How idempotency and race condition handling changes when payment or stock is an external API |

## Common commands

```bash
# infrastructure
docker compose up -d          # start mongo replica set
docker compose down           # stop
docker compose logs mongo     # debug replica set init

# app
npm install
npm run seed                  # seed product with stock=10, prints PRODUCT_ID
npm run start:dev             # watch mode on port 3000

# reset between load test runs
docker compose exec mongo mongosh payment_idem_race --eval "db.dropDatabase()"
npm run seed

# check mongo state after load test
docker compose exec mongo mongosh payment_idem_race \
  --eval "db.products.find({},{name:1,stock:1}).pretty()" \
  --eval "print('paid orders:', db.orders.countDocuments({status:'paid'}))"

# load tests (PRODUCT_ID comes from npm run seed output)
PRODUCT_ID=<id> node loadtests/phase-0-autocannon.js
PRODUCT_ID=<id> k6 run loadtests/phase-0.k6.js
```

## Key concepts established so far

These are decisions and insights that came from discussion — not obvious from the code alone.

- **Idempotency and race conditions are orthogonal problems.** Idempotency fixes "same request sent multiple times." Race condition fixes "different requests modifying the same data." Neither fixes the other. Both are needed.
- **The unique index is the load-bearing part of idempotency**, not application code. The naive `findOne → create` pattern has the same race condition as Phase 0's stock bug. Only the DB insert + duplicate key error closes the gap.
- **Unique constraints are not automatic in any database.** You must explicitly declare them. They must exist before data arrives. NULL handling and sharding behavior differ per database.
- **The payment delay is not test noise.** It simulates real provider network latency (100–500ms in production) and is the mechanism that keeps all 100 requests in-flight simultaneously, making the race reproducible locally.
- **The client owns the idempotency key**, not the server. The client generates a UUID before the first attempt and reuses it on every retry. The server never generates it.
- **Replay must return the same HTTP status code as the original.** Store the full response (status + body), not just a "completed" boolean.

## Important constraints

- **No Redis.** All caching, locking, and queuing uses MongoDB. If you suggest a Redis-based solution, explicitly flag it as an alternative and provide the MongoDB equivalent.
- **Replica set required.** Transactions and change streams need a replica set. The Docker Compose healthcheck auto-initiates `rs0` on first boot. The Mongo URL must include `replicaSet=rs0&directConnection=true` — without `directConnection=true`, the driver tries to discover other nodes and fails on a single-node setup.
- **No strict clean architecture.** Keep modules direct and readable. The learning goal is concurrency mechanics, not layering patterns. Don't add interfaces, abstractions, or extra indirection beyond what already exists.
- **Don't remove the artificial payment delay** in `payments.service.ts`. The 100–300ms sleep is what makes the race condition reliably reproducible on a fast machine.
- **Don't add comments explaining what code does.** Only add comments when something is non-obvious or is a deliberate learning marker (e.g., `// Phase 0: intentionally naive`).

## Code style

- TypeScript strict-ish (strictNullChecks on, noImplicitAny off)
- Mongoose schemas use `@Schema` / `@Prop` decorators
- Services use constructor injection via NestJS DI
- No DTOs or validation pipes yet — raw `@Body()` is intentional for simplicity

---
> Source: [pigman8857/payment-idem-race](https://github.com/pigman8857/payment-idem-race) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
