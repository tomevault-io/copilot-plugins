## banking-transaction-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
make up          # Build and start all Docker services (databases, migrations, both services)
make down        # Stop and remove containers and volumes
make logs        # Follow all service logs
make test        # Run full test suite (npm run test:all)
```

### Build & Lint (from repo root)
```bash
npm run build        # Compile all workspaces via tsc
npm run typecheck    # Type-check without emitting (--noEmit)
npm run lint         # ESLint across all workspaces
```

### Testing
```bash
npm test                      # Jest: unit + integration tests
npm run test:unit             # Jest: __tests__ directories only
npm run test:integration      # Jest: tests/integration/*.test.ts

# Run a single Jest test file
npx jest services/account-service/src/__tests__/routes.test.ts
npx jest services/transfer-service/src/__tests__/saga.test.ts
npx jest tests/integration/transfer.test.ts
```

### Per-workspace commands
```bash
npm run build --workspace @cn-banking/account-service
npm run dev   --workspace @cn-banking/transfer-service
```

## Architecture

**Monorepo** (npm workspaces) with two microservices, each backed by its own PostgreSQL database:

```
services/account-service/   (port 3001, accounts_db on port 5433)
services/transfer-service/  (port 3002, transfers_db on port 5434)
shared/types/               (@cn-banking/shared-types — Account, Transfer, SagaState, DTOs)
infra/db-init/              (SQL migrations, run once at container startup)
tests/integration/          (cross-service integration tests)
Artifacts/                  (PRD, BRD, system design, API contracts, roadmap)
```

TypeScript path alias `@cn-banking/shared-types` resolves to `shared/types/src/index.ts` (configured in `tsconfig.base.json`).

## SAGA Pattern (core logic)

The transfer service implements a **SAGA orchestration** pattern for distributed transactions. The `TransferSaga` class in [services/transfer-service/src/saga.ts](services/transfer-service/src/saga.ts) drives the state machine:

1. Create transfer record (`status: initiated`)
2. `PATCH /v1/accounts/{fromId}/debit` → `status: debited`
3. `PATCH /v1/accounts/{toId}/credit` → `status: completed`

**Compensation:** If credit fails after a successful debit, the SAGA credits the source account back. All state transitions are persisted to a `saga_state` JSONB column for auditability.

The debit/credit endpoints on the account service are **internal** (used only by the SAGA, not exposed to end users).

## Database constraints as business rules

PostgreSQL `CHECK` constraints enforce critical invariants directly in the schema — do not replicate these validations in application code:
- `accounts.balance >= 0` — prevents overdrafts
- `transfers.amount > 0` — prevents zero/negative transfers

The account service returns `422` (mapped from `pg error 23514`) when a debit would violate the balance constraint.

## Inter-service communication

Transfer service calls account service over HTTP using axios. The base URL is configured via `ACCOUNT_SERVICE_URL` (defaults to `http://account-service:3001` inside Docker). The HTTP client is injected into `TransferSaga` for testability — unit tests mock it with `jest.fn()`.

## Environment

Copy `.env.example` to `.env` before running locally. All service ports, DB credentials, and inter-service URLs are configured there. Docker Compose reads this file automatically.

## Testing approach

- **Unit tests** (`__tests__/`): Mock `pg.Pool` and axios; test business logic in isolation.
- **Jest integration tests** (`tests/integration/*.test.ts`): Mock axios and DB pool; test full Express route handling.

Three tests are required for Phase 1 completion:
1. Unit — compensation fires correctly when credit step throws (`saga.test.ts`)
2. Integration — insufficient funds returns 422 and both balances stay unchanged (`transfer.test.ts`)
3. Integration — happy-path transfer updates both balances atomically (`transfer.test.ts`)

## Current phase

Phase 1 is in progress (clean rebuild under Sam ownership). Phases 2–5 (Ledger, Fraud, Notifications, API Gateway, Kafka, Redis, Auth, Kubernetes, Terraform, observability) are deferred.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akasharumugamm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akasharumugamm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
