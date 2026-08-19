## ukss-sms-js-admin

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This is **Stock2 (JS Admin)** - the modern replacement for the legacy **SMS (Stock Management System)**. We are migrating a 15-year-old CakePHP application to NodeJS/ReactJS, module by module.

| System | Name | URL | Technology |
|--------|------|-----|------------|
| Legacy | SMS / Stock System | stock.uksoccershop.com | CakePHP 1.x / PHP 5.6 |
| New | Stock2 / JS Admin | stock2.uksoccershop.com | Fastify + React + TypeScript |

**Migration Approach**: Both systems run in parallel. Features are migrated incrementally. Database is shared.

## Documentation scope

This repository holds **code and code-adjacent docs only**: module READMEs, API contracts, OpenAPI specs, dev setup, deployment mechanics. Synthesis docs, audits, RCAs, integration rationale, decision matrices, and historical-decision write-ups are maintained in a separate documentation repository — they do not belong here. If you are drafting one of those, raise it before opening a PR so it can be redirected to the right home.

## Context Loading (Skills)

For migration work, load relevant context from `.claude/skills/`:

| Skill File | When to Load | Keywords |
|------------|--------------|----------|
| `migration-context.md` | Starting migration work, cron jobs | SSO, migration strategy, phases, cron jobs, CLI |
| `sms-reference.md` | Finding SMS documentation | modules, controllers, documentation index |
| `sms-database.md` | Working with existing tables | tables, schema, relationships |
| `marketplace-integration.md` | Marketplace integrations, new marketplace | Fruugo, Sportdeal, OnBuy, feeds, jsa_MarketplaceListing |
| `coding-guidelines.md` | Code style and conventions | patterns, naming, style |
| `marketplace-orders-integration-guide.md` | Adding new marketplace order integrations | orders, fetch, process, status update |
| `marketplace-price-inventory-sync.md` | Adding new marketplace price & inventory sync | sync, price, inventory, adapter, currency, eBay, Allegro, Debenhams |

**SMS Documentation Path**: `D:\wamp64\www\UKSS-SMS-Production\documentation\`

## Migration Priority

**Cron Job Migration Order** (only enabled jobs, CLI-based implementation):
1. **Orders Integration** - Shopify, Baselinker, Fruugo, OnBuy, Walmart, Toffs, MPlaza orders
2. **Product & Inventory Feeds** - Shopify inventory/price, Baselinker sync, marketplace feeds
3. **Remaining Jobs** - FC integration, NP table, BraveOtter, payments, monitoring

See `.claude/skills/migration-context.md` for complete job listings and schedules.

## Project Structure

```
apps/api/         # Fastify API server (TypeScript)
apps/web/         # React frontend (Vite + Shadcn/ui)
packages/shared/  # Shared types, schemas, utilities
e2e/              # Playwright E2E tests
.claude/skills/   # Context loading for AI agents
```

## Common Commands

```bash
# Development
pnpm dev              # Start all dev servers (API: 4000, Web: 3002)
pnpm build            # Build all packages
pnpm lint             # Lint all code

# Database (PostgreSQL default)
pnpm db:push          # Push schema to database
pnpm db:seed          # Seed initial data
pnpm db:studio        # Open Prisma Studio

# Database (MySQL - for SMS compatibility)
DATABASE_PROVIDER=mysql pnpm db:push
DATABASE_PROVIDER=mysql npx tsx apps/api/prisma/seeds/seed-mysql.ts

# Testing
pnpm test:e2e         # Run Playwright E2E tests
pnpm test:e2e:ui      # E2E with interactive UI

# Docker
docker-compose -f docker-compose.dev.yml up -d
docker-compose -f docker-compose.dev.yml down
```

## Architecture

### Tech Stack
- **Frontend**: React 18, Vite, Shadcn/ui, TanStack Query, Zustand, React Hook Form + Zod
- **Backend**: Fastify, Prisma ORM, PostgreSQL/MySQL, Redis, BullMQ
- **Auth**: JWT with access (memory) + refresh (HTTP-only cookie) tokens

### Backend Module Pattern

**Standard modules:**
```
modules/example/
├── example.routes.ts    # Route definitions & handlers
├── example.service.ts   # Business logic (optional)
└── example.schema.ts    # Zod validation schemas
```

**Note:** Marketplace modules use hyphenated naming (`fruugo-routes.ts`, `sportdeal-routes.ts`). The `marketplaces` module has a complex structure with `adapters/`, `cli/`, `config/`, `db/`, `utils/` subdirectories.

### CLI Infrastructure

Cron jobs are implemented as CLI commands using `commander`:
```bash
pnpm cli <command>         # Run CLI commands (e.g., fetch-shopify-orders)
pnpm sync:marketplace      # Sync marketplace inventory
pnpm sync:all-stores       # Sync all stores
```

| Path | Purpose |
|------|---------|
| `apps/api/src/cli.ts` | CLI entry point (commander setup) |
| `apps/api/src/cli/` | CLI command files (fetch-*-orders, sync-*, update-*-status) |
| `ecosystem.config.cjs` | PM2 production process manager config |

Route handlers use Fastify plugins for auth:
```typescript
// Auth only (no specific permission required):
fastify.get('/', {
  preHandler: [fastify.authenticate],
  handler: async (request, reply) => { ... }
})

// Auth + permission check (colon-separated resource:action):
fastify.get('/', {
  preHandler: [fastify.requirePermission('resource:action')],
  handler: async (request, reply) => { ... }
})
```

### Frontend Patterns
- Pages in `apps/web/src/pages/`
- Data fetching with TanStack Query (`useQuery`, `useMutation`)
- Forms use React Hook Form + Zod validation
- Auth state in Zustand store (`stores/authStore.ts`)
- API client with auto-refresh at `lib/api.ts`

### Shared Package
Import from `@admin/shared`:
- Types: `User`, `Role`, `Permission`, `ApiResponse<T>`, `PaginatedResponse<T>`
- Schemas: `loginSchema`, `createUserSchema`, etc.

## Critical Rules

1. **Server-side DataTables only** - Never implement client-side pagination/sorting/filtering. All DataTable operations must be computed on the API with database queries.

2. **Follow existing patterns** - Match the conventions in existing modules. Small features extend existing modules; large features get new modules.

3. **Zod validation** - All API inputs must be validated with Zod schemas.

4. **Extension points** - Use hooks/plugins for customization instead of modifying core framework code.

5. **SMS Compatibility** - When migrating SMS features, consider database compatibility and SSO requirements.

6. **Verify DB columns before writing raw SQL** - Never assume column names from specs or conceptual descriptions. Before writing any `$queryRaw` / `$queryRawUnsafe` statement, verify every column exists:
   ```sql
   SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS
   WHERE TABLE_SCHEMA='zenc_main_db' AND TABLE_NAME='your_table'
   AND COLUMN_NAME IN ('col1','col2',...);
   ```
   Use `.env.staging` credentials for direct DB access during development. See `documentation/ORDERS-DB-ISSUES-RCA.md` for the full history of issues caused by skipping this step.

7. **Specs must use `table.column` format** - When writing specs that reference database fields, always specify the table name (e.g. `customers.fraud_status`, not just `fraud_status`). Ambiguous references cause implementing agents to put columns on the wrong table.

8. **ZenCart constants ≠ table names** - The legacy SMS uses PHP constants like `TABLE_STORE` which map to actual table names (e.g. `store_management`, not `store`). Always look up the constant definition in `/mnt/2tbdisk/proxmox-home-dir/ukss/UKSS-Production/includes/database_tables.php` before using a table name.

9. **Check the PHP source when migrating** - The legacy orders page is at `/mnt/2tbdisk/proxmox-home-dir/ukss/UKSS-Production/webadmin/orders.php` (10,185 lines). When behavior or column names are unclear in a spec, check the actual PHP query being migrated — don't infer column names from HTML form field names or conceptual descriptions.

## Key Files

| File | Purpose |
|------|---------|
| `apps/api/src/app.ts` | Fastify setup & plugin registration |
| `apps/api/src/config/index.ts` | Environment validation (Zod) |
| `apps/api/prisma/schema.prisma` | Database schema |
| `apps/web/src/App.tsx` | Route definitions |
| `apps/web/src/lib/api.ts` | Axios client with interceptors |
| `apps/web/src/stores/authStore.ts` | Auth state management |
| `apps/api/src/cli.ts` | CLI entry point (commander) |
| `ecosystem.config.cjs` | PM2 production config |

## Environment

Key variables in `.env`:
- `DATABASE_PROVIDER`: `postgresql` (default) or `mysql`
- `DATABASE_URL`: PostgreSQL connection string
- `MYSQL_DATABASE_URL`: MySQL connection string (for SMS database)
- `REDIS_URL`: Redis connection
- `JWT_SECRET`: Min 32 chars, must change from default
- `VITE_API_URL`: API URL for frontend (baked in at build time)

## URLs

| Environment | Frontend | API |
|-------------|----------|-----|
| Development | http://localhost:3002 | http://localhost:4000 |
| Staging | https://stock2-staging.uksoccershop.com | https://stock2-staging.uksoccershop.com/api |
| Production | https://stock2.uksoccershop.com | https://stock2.uksoccershop.com:4000 |

### Staging Server
- **Server IP**: 92.52.120.222
- **DB credentials**: See `.env.staging` (gitignored)
- **Deployment**: GitHub Actions `JsAdmin - deployment` workflow, triggered via `workflow_dispatch` on `staging` branch
- **TODO: Remove `.env.staging` credentials once orders module work is complete**

### Shared Database (CRITICAL)
Both **zenc** (`www.uksoccershop.com/webadmin/`) and **Stock2 staging** (`stock2-staging.uksoccershop.com`) read from the **same MySQL database**. For any given order ID, both systems must display identical data. Any difference is a real bug in Stock2, not a database divergence issue.

## Default Credentials

- Email: `admin@example.com`
- Password: `Admin@123`

---

## Migration Reference

### SSO Requirement
When admin logs into Stock2, they should also be logged into SMS. See `.claude/skills/migration-context.md` for implementation details.

### SMS Documentation Quick Reference

| Topic | SMS Documentation Path |
|-------|------------------------|
| System Overview | `documentation/system_overview.md` |
| Database Schema | `documentation/database/database_schema.md` |
| Authentication | `documentation/authentication/authentication_guide.md` |
| Cron Jobs | `documentation/CRONICLE_CRON_JOBS.md` |
| GRID Module | `documentation/core_modules/grid/` |
| Scanning | `documentation/core_modules/universal_scanning/` |
| Inventory | `documentation/core_modules/inventory/` |
| Shopify | `documentation/integrations/shopify/` |
| Marketplace Orders | `documentation/integrations/marketplace_orders/` |

### Key SMS Tables (for Prisma mapping)
- `sms_users` - User accounts (role: 1=Super Admin, 2=Admin, 3=Freelancer)
- `sms_permissions` - Permission definitions
- `product_lists` - Product codes/models
- `product_quantities` - Size-level inventory
- `box_lists` - Storage boxes
- `teams` - Football clubs/brands

### When Migrating a Feature

1. **Read SMS documentation** - Find relevant guide in `D:\wamp64\www\UKSS-SMS-Production\documentation\`
2. **Check database tables** - See `.claude/skills/sms-database.md`
3. **Review CakePHP controller** - Located at `D:\wamp64\www\UKSS-SMS-Production\app\controllers\`
4. **Design Stock2 implementation** - Follow patterns in this codebase
5. **Consider SSO/compatibility** - Both systems must work during transition

### If Uncertain
Ask before implementing. It's better to clarify than to break the migration workflow.

## SSH Read-Only Access to UKSS Server (92.52.120.222)

- Use SSH config alias: `ukss-claude-ro` (read-only user, direct connection)
- Private key: `~/.ssh/claude_ro_ukss`
- This is a read-only user — can browse filesystem, read logs, check services, but cannot write or modify anything
- Allowed sudo commands: `cat`, `head`, `tail`, `less`, `ls`, `find`, `grep`, `journalctl`, `systemctl status`, `df`, `free`, `ps`, `ss`, `top`, `docker logs/ps/inspect`, `du`, `stat`

---
> Source: [velsof/ukss-sms-js-admin](https://github.com/velsof/ukss-sms-js-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
