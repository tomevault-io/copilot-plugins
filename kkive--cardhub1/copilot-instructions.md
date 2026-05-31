## cardhub1

> **Purpose:** A card management platform for creating, browsing, sharing, and trading digital cards. Supports SillyTavern/TavernAI-compatible character card import/export. Includes free and paid content with secure download workflows. Collections bundle exactly 3 cards (character + worldbook + preset) and export as ZIP.

# AGENTS.md - Cards hub

## Project

**Name:** Cards hub
**Purpose:** A card management platform for creating, browsing, sharing, and trading digital cards. Supports SillyTavern/TavernAI-compatible character card import/export. Includes free and paid content with secure download workflows. Collections bundle exactly 3 cards (character + worldbook + preset) and export as ZIP.

## Architecture

```
cards-hub/
+-- apps/
|   +-- api/          - NestJS backend (REST API + BullMQ worker for exports, cleanup, stats, search-sync)
|   +-- web/          - Umi Max / Ant Design Pro frontend
+-- packages/
|   +-- shared/       - Shared TypeScript types (card schema, SillyTavern compat)
+-- infra/
|   +-- docker/       - Dockerfiles, nginx config
+-- docker-compose.yml
+-- AGENTS.md         - This file (AI collaboration rules)
+-- README.md         - Human setup guide
```

### Stack

| Layer | Technology |
|-------|-----------|
| API | NestJS 10 + Prisma 6 + MySQL 8.4 |
| Queue | BullMQ + Redis 7 |
| Search | Meilisearch |
| Web | Umi Max 4 + Ant Design 5 |
| Auth | JWT + WebAuthn / Passkeys |
| Container | Docker Compose |
| Worker | BullMQ background worker for exports, cleanup, stats, and search index sync (does not serve user pages) |

### Key Directories

- `apps/api/src/` - NestJS source (controllers, services, modules)
- `apps/api/src/auth/` - Auth module (register, login, JWT, `/api/auth`)
- `apps/api/src/card/` - Card CRUD, search, publish/unpublish, admin list
- `apps/api/src/collection/` - Collection CRUD, publish/unpublish, admin list, ZIP export
- `apps/api/src/order/` - Order creation and admin list
- `apps/api/src/file/` - File upload, download, card export
- `apps/api/src/worker/` - BullMQ worker processors (export, cleanup, stats, search-sync)
- `apps/api/src/worker.ts` - BullMQ worker entrypoint (`pnpm start:worker`)
- `apps/api/prisma/` - Prisma schema, migrations, and seed script
- `apps/web/src/pages/cards/` - Mobile-friendly card marketplace (search, filter drawer, detail drawer, paid order creation)
- `apps/web/src/pages/collections/` - Public collection market (search, detail, whole ZIP export, paid order creation)
- `apps/web/src/pages/admin/` - Mobile admin console: overview, cards, collections, orders, config, audit
- `apps/web/src/pages/login/` - Mobile-safe login page (JWT + passkey)
- `apps/web/src/pages/register/` - Mobile-safe registration page
- `apps/web/src/services/api.ts` - Fetch-based API client (JWT from localStorage, `{ data, error }` shape)
- `apps/web/src/layouts/index.tsx` - Sticky top nav bar with mobile hamburger drawer
- `packages/shared/src/` - Shared types and converters

### API Modules

| Module | Path | Description |
|--------|------|-------------|
| Health | `/api/health` | Dependency health checks (DB, Redis, Meilisearch, storage) |
| Auth | `/api/auth` | Register, login, current user (`/api/auth/register`, `/api/auth/login`, `/api/auth/me`) |
| Cards | `/api/cards` | CRUD, public list/search, admin list (`/api/cards/admin/list`), publish/unpublish |
| Tags | `/api/tags` | List and create tags |
| Files | `/api/files` | Upload, download, card export (platform JSON, SillyTavern V2, TavernAI) |
| Admin | `/api/admin` | Bootstrap detection and admin creation |
| Passkey | `/api/passkey` | WebAuthn registration/login challenge and verify |
| Orders | `/api/orders` | Create orders (card or collection), user list, admin list (`/api/orders/admin/list`) |
| Payments | `/api/payments` | Stripe, YiPay, and epay webhook endpoints |
| Collections | `/api/collections` | CRUD, public list/search, admin list (`/api/collections/admin/list`), publish/unpublish, ZIP export |
| Audit | `/api/audit` | Audit log listing and per-action config |
| Users | `/api/users` | Admin user management (list, role update, delete) |
| Config | `/api/config` | System config key-value store |

### Web Routes

| Route | Page | Description |
|-------|------|-------------|
| `/cards` | `pages/cards/index` | Mobile-friendly marketplace: search, filter drawer, sort, grid/table, detail drawer, paid order creation |
| `/collections` | `pages/collections/index` | Public collection market: search, detail view, whole ZIP export, paid order creation |
| `/admin` | `pages/admin/index` | Mobile admin task flow: overview, card management, collection management, order lookup, config, audit |
| `/login` | `pages/login/index` | Mobile-safe auth page (JWT login + passkey), `layout: false` |
| `/register` | `pages/register/index` | Mobile-safe registration page, `layout: false` |

The web app uses a sticky top nav bar layout (`layouts/index.tsx`) with a horizontal menu (cards, collections) and auth buttons. On mobile (< 640px) the nav collapses into a hamburger drawer. The API client wrapper (`services/api.ts`) uses fetch with JWT from localStorage and returns `{ data, error }` for all calls.

## Run Modes

**Mode A — Full Docker:** `docker compose up -d` (or `pnpm docker:up`). Starts mysql, redis, meilisearch, api, worker, web, nginx. Open `http://localhost` or `http://localhost:8000`.

**Mode B — Local Development:** `docker compose up -d mysql redis meilisearch` (or `pnpm start:infra`) + `pnpm dev`. Only infra runs in Docker; API and Web run on the host with hot-reload. **Do not run the full Docker stack and `pnpm dev` at the same time** — ports 3000 (api) and 8000 (web) will conflict.

## Local Commands

```bash
pnpm install               # Install all dependencies
pnpm dev                   # Run api + web in parallel (dev mode)
pnpm dev:api               # Run API only (watch mode)
pnpm dev:web               # Run web only (dev mode)
pnpm build                 # Build all packages (shared -> api + web)
pnpm typecheck             # Type-check web app (tsc --noEmit)
pnpm typecheck:web         # Type-check web app only
pnpm start:api             # Start API in production mode
pnpm start:worker          # Start BullMQ worker
pnpm db:generate           # Generate Prisma client
pnpm db:migrate            # Run Prisma migrations (dev)
pnpm db:migrate:deploy     # Deploy Prisma migrations (prod)
pnpm db:seed               # Seed database with admin, tags, example card
pnpm lint                  # Lint all packages
pnpm test                  # Test all packages
pnpm start:infra           # Start infra containers only (mysql, redis, meilisearch)
pnpm docker:up             # Start all Docker services
pnpm docker:down           # Stop all Docker services
```

## Environment Rules

- All env vars are defined in `.env.example`. Copy to `.env` for local dev.
- `DATABASE_URL` - MySQL connection string for host-local dev (must use mysql:// protocol).
- `REDIS_URL` - Redis connection for host-local dev (BullMQ queues, passkey challenge storage).
- `MEILI_HOST` / `MEILI_API_KEY` - Meilisearch connection for host-local dev.
- **Docker overrides:** `DOCKER_DATABASE_URL`, `DOCKER_REDIS_URL`, `DOCKER_MEILI_HOST` — used by `docker-compose.yml` containers. These default to Docker-network service names (`mysql`, `redis`, `meilisearch`). Set them in `.env` only if you need custom Docker-network URLs. The host-local `DATABASE_URL`/`REDIS_URL`/`MEILI_HOST` (containing `localhost`) are NOT used by containers.
- `DOCKER_STORAGE_DIR` — Storage path inside Docker containers (default `/app/storage`). The host-local `STORAGE_DIR` (e.g. `./storage`) is only used by `pnpm dev` and is NOT injected into containers.
- `PASSKEY_RP_ID` / `PASSKEY_ORIGIN` - WebAuthn relying party config.
  - Local dev: `localhost` / `http://localhost:8000`
  - Production: domain name / `https://domain` (HTTPS required)
- `ADMIN_EMAIL` - Email for the seeded admin user.
- `STORAGE_DIR` - File storage directory for host-local dev (default `./storage`). Not used by Docker containers.
- `DOCKER_STORAGE_DIR` - File storage directory inside Docker containers (default `/app/storage`).
- `STRIPE_SECRET_KEY` - Stripe API secret key for webhook verification.
- `STRIPE_WEBHOOK_SECRET` - Stripe webhook signature verification secret.
- `YIPAY_SECRET` - YiPay MD5 signature secret.
- Never commit `.env` files. Only `.env.example` is tracked.

## Storage Rules

- **Public storage** (`storage/public/`): Files here are freely accessible via API download. Use for free card images, avatars, etc.
- **Paid storage** (`storage/paid/`): NEVER served directly. Paid files are only accessible through API endpoints that verify payment/entitlement.
- **Card exports** (`storage/exports/`): Generated JSON exports for individual cards.
- **Collection exports** (`storage/collection-exports/`): Generated ZIP archives for collections (manifest.json + character.json + worldbook.json + preset.json).
- Host-local dev uses `STORAGE_DIR` env var; Docker containers use `DOCKER_STORAGE_DIR` defaulting to `/app/storage` via the `storage_data` volume.
- The `FileAsset.visibility` field determines which storage path a file uses.

## Data Model Rules

- Cards and collections have a `status` field: `draft` (default for new) or `published`.
- Public list/search/detail endpoints only return `published` items. Admin endpoints can see drafts.
- Collections always bundle exactly 3 cards: one `character`, one `worldbook`, one `preset`.
- Orders have `targetType` (`card` or `collection`) and optional `cardId`/`collectionId`.
- Entitlements are per-user per-card or per-user per-collection (separate unique constraints).
- Purchasing a collection grants collection entitlement only; it does NOT grant individual card entitlements.

## Payment Rules

- Cards and collections with `price > 0` are paid content.
- Paid files must NEVER be exposed via static serving or public URLs.
- Downloads/exports of paid content require: (1) authenticated user, (2) verified entitlement.
- JWT is the preferred auth mechanism. In local/dev environments, pass `X-Dev-User-Id` header to identify the user for entitlement checks when JWT is unavailable. Free content remains anonymous.
- Paid card export/download: `POST /api/files/:cardId/export` and `GET /api/files/:id/download` check entitlement for paid cards.
- Paid collection export/download: `POST /api/collections/:id/export` and `GET /api/collections/exports/:exportId/download` check entitlement for paid collections.
- Orders track: userId, targetType, cardId/collectionId, amount, status (pending/paid/refunded/cancelled).
- Entitlements are created on successful payment events.
- Webhook idempotency: Redis lock + DB unique webhook receipt. Receipt is recorded only after successful payment processing so that retries of failed side-effects are safe.
- Supports: Stripe (checkout.session.completed), YiPay (TRADE_SUCCESS), and epay.

## Worker Queues

| Queue | Processor | Description |
|-------|-----------|-------------|
| `export` | ExportProcessor | Generate JSON exports (platform, SillyTavern V2, TavernAI) |
| `search-sync` | SearchSyncProcessor | Sync cards to Meilisearch index |
| `cleanup` | CleanupProcessor | Remove orphaned exports, old webhook receipts |
| `stats` | StatsProcessor | Recalculate download counts, daily summaries |

## AI Collaboration Rules

- **Read this file first** before making any changes to the codebase.
- **Claude is the sole code editor for this batch cycle.** Codex reviews and verifies only.
- Preserve user-made changes when merging or rebasing.
- All API endpoints must be under the `/api` prefix.
- Use Prisma for all database operations. No raw SQL.
- Shared types must be defined in `packages/shared` and imported via `@cards-hub/shared`.
- NestJS modules follow: controller -> service -> repository pattern.
- Worker tasks use BullMQ queues registered in the API module.
- Frontend routes are defined in `apps/web/.umirc.ts`.
- Do not create fake placeholder implementations that would cause runtime errors.
- Prefer English for code, comments, and docs. Chinese only in user-facing UI text if needed.
- Follow existing file naming conventions: kebab-case for files, PascalCase for classes/types.
- All source files must be UTF-8. Chinese text in UI is acceptable; avoid mojibake (garbled encoding).
- Prisma migrations are tracked in `apps/api/prisma/migrations/`. Always add a new migration directory when changing the schema; never modify existing migration files.

## Acceptance Checks

Before merging any batch:

1. `pnpm install` completes without errors.
2. `pnpm build:shared` compiles the shared package.
3. `pnpm build:api` compiles the API (after prisma generate).
4. `pnpm build:web` compiles the web app.
5. `pnpm typecheck` passes with zero errors.
6. `docker compose config` validates the compose file.
7. `.dockerignore` excludes `node_modules`, `dist`, `.git`, `.env`, caches, and other local artifacts so Docker builds don't tar Windows junctions or ship secrets.
8. No paid files are exposed via nginx `default.conf`.
9. All env vars referenced in code have entries in `.env.example`.
10. AGENTS.md is up to date with any architecture changes.
11. Mobile viewports (390x844, 430x932) show no horizontal overflow on `/cards`, `/collections`, `/admin`, `/login`, `/register`.
12. Admin workflow verified end-to-end: create card draft -> publish -> create collection draft -> publish -> order lookup.

---
> Source: [kkive/cardhub1](https://github.com/kkive/cardhub1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
