## spechive

> SpecHive is a multi-tenant test reporting platform that ingests test results from CI runners, stores artifacts in S3-compatible storage, and presents dashboards. Built with NestJS (backend), React/Vite (dashboard), Drizzle ORM, and PostgreSQL with row-level security.

# SpecHive

SpecHive is a multi-tenant test reporting platform that ingests test results from CI runners, stores artifacts in S3-compatible storage, and presents dashboards. Built with NestJS (backend), React/Vite (dashboard), Drizzle ORM, and PostgreSQL with row-level security.

## Repository structure

```
apps/
  gateway/         NestJS API gateway — JWT auth, rate limiting, proxy (port 3000)
  ingestion-api/   NestJS API — receives reporter events (port 3001, internal)
  worker/          NestJS worker — processes outbox events (port 3002, internal)
  query-api/       NestJS API — serve data to the dashboard (port 3003, internal)
  dashboard/       React/Vite SPA (port 5173)

packages/
  api-types/             Typed API response interfaces
  database/              Drizzle schema, migrations, seed, connection helpers
  eslint-config/         Shared ESLint flat config
  nestjs-common/         Shared NestJS modules (config, filters, health)
  playwright-reporter/   Playwright reporter for SpecHive
  reporter-core-protocol/ Protocol types for test reporters
  shared-types/          Branded ID types and enums (shared across all packages)
  typescript-config/     Shared tsconfig bases
```

## Key files

| File                                  | Purpose                                                     |
| ------------------------------------- | ----------------------------------------------------------- |
| `packages/database/src/connection.ts` | `setTenantContext()` for RLS, `createDbConnection()`        |
| `packages/database/src/schema/*.ts`   | Drizzle schema definitions                                  |
| `packages/database/src/migrate.ts`    | `runMigrations()` — bootstrap, outboxy DDL, Drizzle, grants |
| `test/integration-global-setup.ts`    | Seeds test data (org, project, token, user)                 |
| `commitlint.config.js`                | Conventional commit rules                                   |
| `.env.example`                        | Full environment variable reference                         |

## Prerequisites

- Node.js >= 22
- pnpm >= 9
- Docker & Docker Compose (for Postgres, MinIO, Outboxy)

## Quick start

```bash
cp .env.example .env
pnpm install
pnpm build
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

## Development workflow

### Build & type checking

```bash
pnpm build          # Build all packages and apps
pnpm typecheck      # TypeScript check all packages
```

### Linting & formatting

```bash
pnpm lint           # ESLint with zero warnings
pnpm format         # Format with Prettier
pnpm format:check   # Check formatting without writing
```

### Testing

```bash
pnpm test:unit      # Unit tests (no Docker required)
pnpm test:integration # Integration tests (requires Docker)
pnpm test           # All tests (unit then integration)
```

Integration tests use `test/vitest.integration.config.ts`, run sequentially (`maxWorkers: 1`) with 30s timeout. The `globalSetup` hook seeds test data automatically.

### Database

```bash
pnpm db:generate    # Generate Drizzle migrations from schema
pnpm db:migrate     # Apply pending migrations
pnpm db:seed        # Seed database with initial data
```

### Docker

```bash
pnpm docker:up      # Start all services
pnpm docker:down    # Stop all services
```

Local development uses both compose files: `docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d`

## Database architecture

### Two-role model

| Role                      | Connection string                                              | Purpose                                       |
| ------------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| `spechive` (superuser)    | `postgres://spechive:spechive@localhost:5432/spechive`         | Migrations, seeding, admin ops — bypasses RLS |
| `spechive_app` (app role) | `postgres://spechive_app:spechive_app@localhost:5432/spechive` | Application queries — subject to RLS policies |
| `outboxy`                 | `postgres://outboxy:outboxy@localhost:5432/spechive`           | Outboxy transactional outbox                  |

Both app roles are created by `runMigrations()` in `packages/database/src/migrate.ts` (idempotent — safe to re-run).

### Row-Level Security (RLS)

All tenant-scoped tables have RLS policies that filter rows by `app.current_organization_id`. The application **must** set this context per transaction:

```typescript
import { setTenantContext } from '@spechive/database';

await db.transaction(async (tx) => {
  await setTenantContext(tx, organizationId);
  // All queries in this transaction are now scoped to the organization
});
```

Without `setTenantContext`, queries return zero rows (fail-closed).

### Schema hierarchy

```
organizations ─→ projects ─→ runs ─→ suites ─→ tests ─→ artifacts
                  │                               │
                  └─→ project_tokens               └─→ test_attempts

users ─→ memberships ←─ organizations

auth: refresh_tokens
```

Tenant boundary is at the organization level. RLS propagates down through foreign key joins.

### Test attempts & retry model

- `test_attempts` stores per-retry-attempt data (status, error, duration); always one row per attempt including the first
- `artifacts.retryIndex` associates artifacts with specific attempts; joined to `test_attempts` via `(testId, retryIndex)` compound match (not FK)
- `tests` table retains summary fields as a denormalized cache; `errorMessage` = "representative error" (last failed attempt for flaky, final attempt otherwise)
- `test_status` enum is shared: attempts only use `passed`/`failed`/`skipped` values

## Environment variables

See `.env.example` for the full list with comments. Key variables:

### PostgreSQL

| Variable                              | Purpose                                     |
| ------------------------------------- | ------------------------------------------- |
| `DATABASE_URL`                        | App-role connection string (subject to RLS) |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | Superuser credentials (migrations, Docker)  |
| `SPECHIVE_APP_PASSWORD`               | Password for `spechive_app` role            |
| `OUTBOXY_PASSWORD`                    | Password for `outboxy` role                 |

### Authentication

| Variable                 | Purpose                                      |
| ------------------------ | -------------------------------------------- |
| `TOKEN_HASH_KEY`         | Argon2 token hashing key (min 32 chars)      |
| `JWT_SECRET`             | JWT signing key (min 64 chars in production) |
| `JWT_ACCESS_EXPIRES_IN`  | Access token expiry (default: 15m)           |
| `JWT_REFRESH_EXPIRES_IN` | Refresh token expiry (default: 7d)           |

### Storage (MinIO)

| Variable                                        | Purpose                                   |
| ----------------------------------------------- | ----------------------------------------- |
| `MINIO_ENDPOINT`                                | S3-compatible storage endpoint (internal) |
| `MINIO_PUBLIC_ENDPOINT`                         | Public endpoint for presigned URLs        |
| `MINIO_BUCKET`                                  | Artifact storage bucket                   |
| `MINIO_APP_ACCESS_KEY` / `MINIO_APP_SECRET_KEY` | MinIO app credentials                     |

### Webhooks & Outboxy

| Variable         | Purpose                                         |
| ---------------- | ----------------------------------------------- |
| `WEBHOOK_SECRET` | Outboxy webhook authentication (min 32 chars)   |
| `WORKER_URL`     | Worker base URL (webhook path appended in code) |

### Dashboard

| Variable       | Purpose                                  |
| -------------- | ---------------------------------------- |
| `VITE_API_URL` | Gateway URL (Vite inlines at build time) |

## Project-specific conventions

- **Branded IDs**: All entity IDs use UUIDv7 with TypeScript branded types (`OrganizationId`, `ProjectId`, etc.) to prevent accidental mixing. Cast with `asProjectId(str)`.
- **Conventional commits**: Enforced by commitlint — `feat:`, `fix:`, `chore:`, etc.
- **Drizzle ORM**: Schema defined in `packages/database/src/schema/`. Migrations in `packages/database/drizzle/`.

## Testing conventions

### Test architecture

```
test/
├── helpers/
│   ├── api-clients/        # Typed HTTP clients (QueryApiClient, IngestionApiClient)
│   ├── factories/          # Event & webhook payload factories
│   ├── assertions/         # Domain-specific assertions
│   ├── wait.ts             # waitForService(), waitForRow(), poll()
│   ├── database.ts         # buildSuperuserDatabaseUrl(), createPostgresConnection()
│   ├── load-dot-env.ts     # Shared .env parser
│   ├── constants.ts        # Seeded IDs, URLs, credentials
│   └── index.ts            # Barrel export
├── unit-helpers/
│   ├── handler-context.ts  # createHandlerContext() for worker handler specs
│   ├── drizzle-mock.ts     # createMockInsertChain(), createMockDb()
│   ├── mock-guards.ts      # MockProjectTokenGuard, MockThrottlerGuard, MockGatewayTrustGuard
│   ├── nestjs.ts           # createTestModule() wrapper
│   └── index.ts
├── integration/            # Integration tests (require Docker stack)
├── integration-global-setup.ts
└── vitest.integration.config.ts
```

### Patterns

- **AAA structure**: Arrange–Act–Assert with blank line separation
- **API Client Objects**: Use `QueryApiClient` / `IngestionApiClient` instead of raw `fetch()`
- **Factories**: Use `createRunStartEvent()`, `createFullRunEvents()` etc. from `test/helpers/factories`
- **Shared wait helpers**: Use `waitForService()`, `waitForRow()`, `poll()` from `test/helpers/wait`
- **Constants**: Use `SEED_ORG_ID`, `SEED_PROJECT_ID` etc. from `test/helpers/constants`
- **Three-layer rule**: Never duplicate helpers — integration tests import from `test/helpers/`, unit tests from `test/unit-helpers/`

### Import paths

```typescript
// Integration tests — all normal-flow tests go through GATEWAY_URL
import { waitForService, SEED_ORG_ID, GATEWAY_URL, QueryApiClient } from '../../helpers';

// Unit tests (handler specs)
import { createHandlerContext } from '../../../test/unit-helpers';

// Unit tests (controller specs)
import { MockGatewayTrustGuard } from '../../../test/unit-helpers/mock-guards';
```

## Important rules

- **NestJS apps compile to CJS**: Despite packages using ESM, the NestJS apps (`ingestion-api`, `worker`, `query-api`) produce CommonJS output via their build step. Do not add `"type": "module"` to their `package.json`.
- **Migrations use the superuser role**: Never run `db:migrate` with `DATABASE_URL` pointing to `spechive_app` — the app role lacks DDL permissions.
- **Seeding uses the superuser role**: The seed script (`pnpm db:seed`) must connect as the superuser to bypass RLS. It accepts `SEED_DATABASE_URL` or falls back to `DATABASE_URL`.
- **Role creation is idempotent**: `runMigrations()` creates/updates `spechive_app` and `outboxy` roles on every run. No manual role management needed.
- **Build order matters**: `shared-types` → `reporter-core-protocol` → `database` → `nestjs-common` → apps. `pnpm build` handles this via workspace topology.
- **Two-file compose strategy**: `docker-compose.yml` is the production base (no host port bindings). `docker-compose.dev.yml` adds host ports, hot-reload, and dev settings. Local development always uses both files.
- **No backward compatibility required**: The platform has not been released yet. All components (reporter, protocol, worker, dashboard) can be changed in lockstep. No need to maintain dual-path support or deprecation shims when refactoring.

---
> Source: [SpecHive/SpecHive](https://github.com/SpecHive/SpecHive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
