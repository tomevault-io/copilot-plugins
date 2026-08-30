## my-portfolio-be

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
yarn start:dev          # Dev server with watch mode (port 3023)
yarn build              # Compile TypeScript to dist/

# Code quality
yarn lint               # ESLint with auto-fix
yarn format             # Prettier formatting

# Testing
yarn test               # Unit tests
yarn test:watch         # Unit tests in watch mode
yarn test:cov           # Coverage report
yarn test:e2e           # End-to-end tests
yarn test -- --testPathPattern=<file>  # Run a single test file

# Database
yarn typeorm migration:run      # Apply pending migrations
yarn typeorm migration:revert   # Revert last migration
yarn typeorm migration:generate -- src/database/migrations/<Name>  # Generate migration from entity changes
```

## Environment Setup

Copy `.env.example` to `.env` and fill in values. Required variables:
- `DB_*` — PostgreSQL connection (host, port, username, password, name)
- `REDIS_*` — Redis connection (host, port, password) used by BullMQ and auth/file caching
- `JWT_SECRET_KEY`, `JWT_EXPIRATION` — JWT auth config
- `SMTP_*` — Gmail SMTP credentials for sending emails
- `FE_APP_URL` — Frontend URL used in email verification links
- `ADMIN_PASSWORD` — Seeded admin account password
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_BUCKET` — S3 file upload/storage

Start dependencies with Docker: `docker-compose up -d db redis`

Swagger UI is available at `http://localhost:3023/api` during development.

## Module Conventions

### Module responsibility & folder structure

Each module is only responsible for its own code. If resource B belongs to resource A, create module B as a sub-folder inside module A's folder, then import it into the parent module.

```
modules/skill-category/               ← parent module (app.module imports this)
├── skill-category.module.ts
├── skill-category.controller.ts
├── skill-category.service.ts
├── dto/
└── skill/                            ← child module lives inside parent folder
    ├── skill.module.ts
    ├── skill.controller.ts
    ├── skill.service.ts
    └── dto/
```

### Response key naming

Response keys must always be the **camelCase of the module name** — never generic words like `data`, `item`, `result`, `category`.

```typescript
// module: skill-category
return { skillCategories, total };   // list
return { skillCategory };            // detail

// module: skill
return { skills, total };            // list
return { skill };                    // detail
```

### Public routes

All routes decorated with `@PermitAll()` must be defined in the single top-level `public` module (`src/modules/public/`). Do not scatter public endpoints across individual resource modules.

`PublicModule` imports the resource modules whose services it needs. Those modules must export the relevant service. `app.module.ts` imports `PublicModule` alongside the other modules.

Public service methods live on the owning resource's service and are named `publicList()`, `publicDetail()`, etc.

## Code Style

Prettier: double quotes (`singleQuote: false`), trailing commas (`trailingComma: all`), 120-char line width. Run `yarn lint && yarn format` before committing.

## Domain Reference (Auto-loaded Skills)

The following skills live in `.claude/skills/` and are automatically injected into context when the relevant domain is detected. You can also invoke them manually as slash commands (e.g. `/arch`).

| Skill | Activates when working on… |
|-------|---------------------------|
| `project-arch` | Module structure, utilities, DTOs, response keys, public routes |
| `nestjs-auth` | Guards, JWT, `@Auth`, `@RequiredPermission`, RBAC, Redis cache keys |
| `nestjs-db` | Entities, TypeORM relations, migrations, `@Transactional()` |
| `s3-file-upload` | File upload, presigned URLs, `FileEntity`, S3 object lifecycle |
| `nestjs-mail-queue` | Email sending, BullMQ, `MailService`, `MailProcessor` |
| `nestjs-concurrency` | Mutex, semaphore, race conditions, `RateLimiter` |
| `nestjs-errors` | `AppException`, `ErrorCode`, exception filters, `LoggerService` |

---
> Source: [hyudoo/my_portfolio_be](https://github.com/hyudoo/my_portfolio_be) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
