## tanstack-start-elysia-better-auth-bun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack monorepo using Bun workspaces with TanStack Start frontend and Elysia backend. The project demonstrates clean architecture with feature-based organization.

### Tech Stack

- **Frontend**: TanStack Start (React Router + SSR), TanStack Query, Jotai (state), shadcn/ui
- **Backend**: Elysia.js with Better Auth, Swagger docs
- **Database**: Drizzle ORM with SQLite/Turso
- **Runtime**: Bun
- **Styling**: Tailwind CSS

## Common Commands

### Development

```bash
# Start all apps in dev mode
bun dev

# Install dependencies
bun install

# Run specific app
bun --filter=backend dev
bun --filter=web dev
```

### Code Quality

```bash
# Type check all packages
bun typecheck

# Lint all packages
bun lint
bun lint:fix

# Format all packages
bun format
bun format:fix

# Check workspace dependencies
bun lint:ws
```

### Database Operations

```bash
# Generate migrations
bun db:generate

# Push schema to database
bun db:push

# Seed database
bun db:seed

# Open Drizzle Studio
bun db:studio
```

### Build & Clean

```bash
# Build all packages
bun build

# Clean all node_modules and build artifacts
bun clean
```

## Architecture

### Monorepo Structure

```
apps/
├── backend/          # Elysia.js API server
└── web/              # TanStack Start frontend
packages/
├── backend-client/   # Type-safe API client (Eden Treaty)
└── db/              # Drizzle ORM schemas and migrations
tooling/             # Shared configs (ESLint, Prettier, TypeScript, Tailwind)
```

### Clean Architecture Layers

- **Domain**: Business entities, repository interfaces (`*-entity.ts`, `*-repository.ts`)
- **Application**: Services, use cases (`*-service.ts`)
- **Infrastructure**: Repository implementations (`*-drizzle-db-repo.ts`, `*-api-repo.ts`)
- **Presentation**: Controllers, components (`*-controller.ts`, `*-routes.ts`)

### Feature Organization

Both frontend and backend use feature-based organization:

```
features/
└── feature-name/
    ├── _components/     # Feature-specific UI components
    ├── _controllers/    # State management controllers
    ├── _domain/         # Domain models and interfaces
    ├── _lib/           # Infrastructure implementations
    └── _services/      # Business logic services
```

## Key Patterns

### Backend (Elysia)

- Use Elysia instances as controllers with method chaining
- Group routes with prefixes and include Swagger `detail` property
- Class-based services with arrow function methods
- All methods accept/return objects for consistency

### Frontend (TanStack Start)

- Class-based controllers with Jotai atoms for state
- All hooks return objects (even single values): `return { isPending }`
- Repository pattern for data fetching with descriptive naming
- Component composition following shadcn/ui patterns

### Authentication

- Better Auth integration on both frontend and backend
- Session management with type-safe exports
- Authentication middleware in backend

### Database

- Drizzle ORM with type-safe schemas
- Repository pattern for data access
- Migrations managed via Drizzle Kit

## Development Guidelines

### Code Style

- Use arrow functions for class methods
- Object parameters and returns for all functions
- Absolute imports with `~` prefix
- Direct imports (avoid barrel files)
- Feature-specific folders use underscore prefix

### File Naming

- kebab-case for all files and directories
- Descriptive repository names: `entity-infrastructure-type-repo.ts`
- Clear entity/model separation: `*-entity.ts` vs `*-model.ts`

### Performance

- Small, focused components with isolated re-renders
- Single-responsibility hooks
- Proper TypeScript with interfaces over types

## Environment Setup

The project uses a shared `.env` file at the root level. Backend and web apps access it via `with-env` scripts that load environment variables.

## Key Features Implemented

- **Count**: File-based persistence demo
- **Messages**: CRUD operations with database persistence
- **Pokemon**: External API integration with caching
- **Auth**: Complete authentication flow with Better Auth

---
> Source: [masrurimz/tanstack-start-elysia-better-auth-bun](https://github.com/masrurimz/tanstack-start-elysia-better-auth-bun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
