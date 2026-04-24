## sdd-fullstack-todo-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Development Commands

### Root Level Commands

- `yarn dev` - Start all applications in parallel (API on :3000, Web on :4321)
- `yarn build` - Build all applications using Turbo
- `yarn test` - Run all tests across workspaces
- `yarn lint` - Run ESLint with auto-fix across all workspaces
- `yarn typecheck` - Type check all TypeScript code
- `yarn format` - Format code with Prettier

### API-Specific Commands (apps/api)

- `yarn workspace @app/api dev` - Start NestJS API in watch mode
- `yarn workspace @app/api test` - Run Jest tests
- `yarn workspace @app/api test:watch` - Run tests in watch mode
- `yarn workspace @app/api prisma generate` - Generate Prisma client
- `yarn workspace @app/api prisma migrate dev` - Run database migrations
- `yarn workspace @app/api prisma studio` - Open Prisma Studio

### Web-Specific Commands (apps/web)

- `yarn workspace @app/web dev` - Start Astro dev server
- `yarn workspace @app/web test` - Run Vitest tests
- `yarn workspace @app/web test:watch` - Run tests in watch mode

## Architecture Overview

This is a **monorepo** using **Yarn workspaces** and **Turbo** for build orchestration.

### Backend (apps/api) - NestJS with Clean Architecture

- **Domain Layer**: Entities, value objects, interfaces, domain services
- **Application Layer**: Use cases, DTOs, mappers, application services
- **Infrastructure Layer**: Controllers, repositories, external services

The API follows strict Clean Architecture principles:

- Domain entities are in `src/{module}/domain/entities/`
- Use cases are in `src/{module}/application/use-cases/`
- Controllers are in `src/{module}/infrastructure/controllers/`
- Repository implementations are in `src/{module}/infrastructure/repositories/`

**Database**: Uses Prisma ORM with SQLite for development. Schema is in
`apps/api/prisma/schema.prisma`.

### Frontend (apps/web) - Astro with React

- **File-based routing**: Pages in `src/pages/`
- **React components**: Reusable components in `src/components/`
- **Static generation**: Pre-built pages with hydration
- **Styling**: TailwindCSS for utility-first styling

### Shared Packages (packages/)

- `@packages/shared-types` - Common TypeScript types
- `@packages/eslint-config` - Shared ESLint configuration
- `@packages/ts-config` - Shared TypeScript configuration

## Key Technologies

- **Backend**: NestJS, Prisma, SQLite, JWT, Jest
- **Frontend**: Astro, React, TailwindCSS, Vitest
- **Tools**: TypeScript, Turbo, ESLint, Prettier, Husky

## Environment Setup

API requires `.env` file in `apps/api/` with:

```text
DATABASE_URL="file:./dev.db"
JWT_SECRET="your-secret"
JWT_EXPIRES_IN="24h"
SESSION_TTL="86400"
```

## Testing Strategy

- **API**: Jest for unit/integration tests, comprehensive test coverage for all layers
- **Web**: Vitest with React Testing Library for component tests
- **E2E**: Not yet implemented

Run `yarn test` to execute all tests or use workspace-specific commands for targeted testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TituxMetal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
