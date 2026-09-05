## local-app

> Localmart is an open source, community-governed local retail marketplace.

# Localmart — Claude Code Context

## What this project is

Localmart is an open source, community-governed local retail marketplace.
It connects independent local businesses with shoppers in their own neighborhoods.
The platform is multi-tenant: each "community" is a named geographic area with its own
set of merchants and volunteer moderators. See `docs/decisions/` for architectural context.

## Stack

| Layer             | Technology                                    |
|-------------------|-----------------------------------------------|
| Consumer frontend | Next.js 14 App Router (`apps/web`)            |
| Backend API       | Hono.js on Node.js (`apps/api`)               |
| Shared types      | `packages/types` — no framework deps          |
| Database ORM      | Drizzle + `packages/db` — source of truth     |
| Database          | PostgreSQL 16 + PostGIS (Docker locally)      |
| Search            | Meilisearch (Docker locally)                  |
| Queue             | BullMQ + Redis (Docker locally)               |
| Object storage    | MinIO locally → AWS S3 in staging/prod        |
| Secrets           | `.env` locally → HashiCorp Vault in cloud     |
| IaC               | Terraform (`infra/`)  — AWS target            |
| Monorepo          | Turborepo + pnpm workspaces                   |

Mobile (`apps/merchant`) is planned but currently excluded from the workspace.

## Project structure

```
apps/web          Next.js consumer frontend
apps/api          Hono.js API — all business logic lives here
apps/merchant     Expo merchant app (excluded from workspace until Tier 5)
packages/types    Domain types shared across all apps — edit this first
packages/db       Drizzle schema + migrations — the schema is the source of truth
packages/config   Shared ESLint + TypeScript base configs
docs/decisions/   Architecture Decision Records (ADRs)
docs/policies/    Community policy documents
infra/            Terraform modules for AWS deployment
```

## Development setup

```bash
make setup   # first time only
make dev     # start all apps
make up      # start Docker services only
```

See `Makefile` for all available commands.

## Key conventions

- **TypeScript strict mode** everywhere — no `any`, no skipping type errors
- **`packages/types`** is the source of truth for domain types; changes there may require schema updates in `packages/db`
- **`packages/db/src/schema.ts`** mirrors `packages/types` exactly — keep them in sync
- **No business logic in packages** — `packages/types` and `packages/db` are pure data/schema, not logic
- **All mutations go through `apps/api`** — the web frontend never writes directly to the DB
- **pnpm only** — never use npm or yarn
- **Zod for all external input validation** at API boundaries
- **Drizzle for all DB access** — no raw SQL except in migrations
- Environment variables: `.env` for local, Vault for cloud. Never commit secrets.

## Database

- Run `make db-generate` after editing `packages/db/src/schema.ts`
- Run `make db-migrate` to apply pending migrations
- Run `make db-seed` to insert dev fixtures (idempotent)
- Run `make db-reset` to wipe and re-seed from scratch

## What NOT to do

- Do not add `@localmart/db` as a dependency in `apps/web` — the frontend never touches the DB
- Do not add new columns to the DB without also adding them to the corresponding type in `packages/types`
- Do not hardcode port numbers — always read from env vars (`API_PORT`, etc.)
- Do not commit `.env` — it is in `.gitignore`
- Do not add `console.log` to production code — use structured logging via a logger (to be added in Tier 1)

## Cloud deployment (future)

Terraform lives in `infra/`. The dev environment mirrors production:

| Local              | AWS (staging/prod)              |
|--------------------|----------------------------------|
| Docker Postgres    | Aurora PostgreSQL 16 + RDS Proxy |
| Docker Redis       | ElastiCache Redis 7              |
| Docker Meilisearch | Meilisearch on ECS Fargate       |
| MinIO              | AWS S3                           |
| `.env` file        | HashiCorp Vault on ECS Fargate   |

See `docs/ROADMAP.md` for the full build order.

---
> Source: [project-local-mart/local-app](https://github.com/project-local-mart/local-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
