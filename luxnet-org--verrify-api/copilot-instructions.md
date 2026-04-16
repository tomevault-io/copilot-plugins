## verrify-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Verrify API is a NestJS 11 backend service for property and company verification, built with TypeScript, TypeORM (PostgreSQL + PostGIS), RabbitMQ, and Cloudinary.

## Common Commands

```bash
npm run start:dev          # Dev server with hot-reload
npm run build              # Compile to dist/
npm run start:prod         # Run migrations then start server
npm run lint               # ESLint with auto-fix
npm run format             # Prettier
npm run test               # Unit tests (Jest)
npm run test:e2e           # E2E tests
npm run test:cov           # Coverage report

# Migrations
npm run typeorm:migration-generate --name=MigrationName
npm run typeorm:migration-create --name=MigrationName
npm run typeorm:migration-run
npm run typeorm:migration-revert
```

## Infrastructure (docker-compose)

`docker-compose up` starts PostgreSQL (PostGIS, port 5432), RabbitMQ (ports 5672/15672), and MailHog (SMTP 1025, UI 8025). Default credentials: user `nest`, password `nest1234`.

## Architecture

Single NestJS module (`AppModule`) with a flat service/controller layout — no feature modules.

- **`src/controller/`** — HTTP endpoints. Admin-specific controllers are prefixed `admin-*`.
- **`src/service/`** — Business logic, organized by domain in subdirectories (auth, user, payment, email, rabbitmq, etc.).
- **`src/model/entity/`** — TypeORM entities (PostgreSQL).
- **`src/model/dto/`, `request/`, `response/`** — Validated DTOs using `class-validator` and `class-transformer`.
- **`src/model/enum/`** — Shared enums (roles, statuses, file types).
- **`src/common/guards/`** — `AuthGuard` (JWT) and `RoleGuard` (RBAC), both registered globally via `APP_GUARD`.
- **`src/common/decorator/`** — `@Public()` to skip auth, `@Roles()` for RBAC, Swagger helpers.
- **`src/config-module/`** — Typed config (Joi-validated env vars) via `@nestjs/config`.
- **`src/config/typeorm.config.ts`** — TypeORM CLI datasource for migrations.
- **`src/database/migrations/`** — TypeORM migrations. Always commit migrations alongside the entity changes that require them.
- **`src/templates/`** — Pug email templates.

### Key patterns

- **Auth**: JWT access tokens + refresh tokens. Global `AuthGuard` protects all routes by default; use `@Public()` to opt out. `@Roles()` + `RoleGuard` for RBAC.
- **Email**: Dual provider — SMTP (via `@nestjs-modules/mailer`) or Resend, selected by `EMAIL_PROVIDER` env var. Email dispatch is event-driven through `EmailEvent` / `ContactEventService` which publish to RabbitMQ.
- **Payments**: Paystack integration with order/transaction entities and webhook handling.
- **Files**: Uploaded to Cloudinary via `CloudinaryProvider`; `FileService` supports transactional operations with an optional `EntityManager`.
- **Validation**: Global `ValidationPipe` with `whitelist: true` and `forbidNonWhitelisted: true`. DTOs must use class-validator decorators.
- **Global prefix**: All routes are prefixed (see `AppConstants.APP_GLOBAL_PREFIX`).
- **Synchronize**: TypeORM `synchronize` is enabled in dev only. In production, use migrations.
- **Swagger**: Available at `/docs` when the server is running.

## Coding Conventions

- Conventional Commits: `feat:`, `fix:`, etc.
- File naming: `*.controller.ts`, `*.service.ts`, `*.entity.ts`, `*.enum.ts`, DTOs as `*-request.dto.ts` / `*-response.dto.ts`.
- PascalCase for classes/enums, camelCase for methods/properties, kebab-case for migration filenames.
- 2-space indent, single quotes, trailing commas (enforced by Prettier).
- ESLint: `@typescript-eslint/no-explicit-any` is off; `no-floating-promises` and `no-unsafe-argument` are warnings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Luxnet-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
