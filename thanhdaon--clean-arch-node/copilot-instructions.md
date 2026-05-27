## clean-arch-node

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Task Management API built with **Clean Architecture** and **Domain-Driven Design (DDD)**. Role-based access with Employers (create/assign tasks) and Employees (view/update assigned tasks).

## Key Commands

```bash
# Development
bun dev               # Start dev server with Vite
bun test              # Run all tests (bun:test)
bun typecheck         # TypeScript type checking

# Database (Neon PostgreSQL)
bun db:generate       # Generate migrations from schema changes
bun db:migrate        # Run pending migrations
bun db:push           # Push schema directly (dev only)
bun db:seed           # Seed database
bun db:studio         # Open Drizzle Studio GUI

# Auth
bun auth:schema-generate  # Generate better-auth schema files
```

## Architecture

### Layer Structure
```
src/
├── domain/          # DDD core - entities, value objects (NO external imports)
├── app/             # Application services - Commands & Queries (CQRS)
├── adapters/        # Infrastructure - repository implementations
├── ports/           # HTTP entry points - Hono handlers
├── common/          # Shared utilities, config, instrumentation
└── db/              # Drizzle schema, migrations, seed
```

### Dependency Flow (Critical Rule)
Dependencies flow inward only: `Domain ← App ← Adapters/Ports`
- **Domain**: Pure business logic, NO imports from other layers
- **App**: Orchestrates domain + defines repository interfaces
- **Adapters**: Implements interfaces from app layer
- **Ports**: HTTP handlers, maps requests to app commands/queries

### Key Patterns

**Domain Entities** (`src/domain/`): Factory functions like `buildMakeTask({ Id })` create entities with encapsulated business rules. Entities expose getters and mutation methods.

**CQRS** (`src/app/command/`, `src/app/query/`): Commands mutate state, queries read. Each exports a factory function that accepts repository dependencies.

**Repository Interfaces** (`src/app/command/adapters.ts`): Defined in app layer, implemented in adapter layer. This enables dependency inversion.

**HTTP Handlers** (`src/ports/http/handlers/`): Each handler exports `route` (OpenAPI definition) and `makeHandler(app)` function.

**App Composition** (`src/server.ts`): Wires repositories → commands/queries → handlers. Single source of truth for dependency injection.

## Adding a New Feature

1. **Domain**: Add entity/logic in `src/domain/` with factory function
2. **App**: Create command/query in `src/app/`, define interface in `adapters.ts` if needed
3. **Adapter**: Implement repository method in `src/adapters/`
4. **Port**: Add HTTP handler with Zod schema in `src/ports/http/handlers/`
5. **Wire**: Register in `src/server.ts` and add route to router

## Testing

Tests use `bun:test` framework with `.spec.ts` suffix.

- **Domain tests**: Pure unit tests with mocked dependencies
- **Adapter tests**: Integration tests against real database

```typescript
// Domain test example - mock the Id dependency
const mockId: ID = {
  newId: mock(() => "mocked-uuid"),
  isValidId: mock((id: string) => id.length > 0),
};
const makeTask = buildMakeTask({ Id: mockId });
```

## Tech Stack

- **Runtime**: Bun
- **Framework**: Hono with OpenAPI/Swagger
- **Database**: Neon PostgreSQL (serverless) with Drizzle ORM
- **Auth**: better-auth with email/password
- **Validation**: Zod schemas
- **Observability**: OpenTelemetry tracing
- **Path alias**: `~` maps to `src/`

## Environment Variables

Required in `.env`:
- `DB_URL` - Neon PostgreSQL connection string
- `JWT_SECRET` - JWT signing secret
- `BETTER_AUTH_SECRET` - Auth library secret
- `BETTER_AUTH_URL` - Auth callback URL
- `OTLP_TRACE_EXPORTER_URL` - OpenTelemetry collector endpoint
- `SERVICE_NAME` - Service identifier for tracing

## API Documentation

Swagger UI available at `/api/doc` when server is running.

---
> Source: [thanhdaon/clean-arch-node](https://github.com/thanhdaon/clean-arch-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
