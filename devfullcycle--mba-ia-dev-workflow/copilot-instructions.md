## project-guidelines

> Project-wide guidelines for the CodeLearn Platform LMS. Covers project structure, tech stack, commands, hexagonal architecture, and key documentation references.


# Project Guidelines

## Project Overview

CodeLearn Platform — an online learning management system (LMS) built as a greenfield monolithic modular application with hexagonal architecture (ports & adapters). Documentation is in Portuguese.

**Docs are in Portuguese**, but everything inside the project (code, function names, file names, variables, comments, API responses, entity names, field names) is in English.

## Project Structure

```
/src
  /<module-name>
    index.ts            # module entry point (exports controllers, use cases, etc.)
    /domain
      /entities         # entities, value objects and aggregates
      /services         # domain services (stateless business logic that doesn't fit in entities or use cases)
      /ports
        /driven         # interfaces for incoming interactions (e.g., controllers)
        /driving        # interfaces for outgoing interactions (e.g., repositories)
    /controllers        # Express route handlers (map HTTP requests to use cases)
    /models             # TypeORM entities and data source configuration
    /repositories       # implementations of driving ports (e.g., TypeORM repositories)
    /services           # application services (use cases, orchestrating domain logic and interactions)
    /container          # dependency injection setup for the module
  /shared
    /utils              # utility functions, helpers, etc.
    /errors             # custom error classes
    /middleware          # Express middleware (e.g., auth, error handling)
      errorHandler.ts   # global error handling middleware
    /config             # configuration files and environment variable handling
    /migrations         # database migration files
    container.ts        # global dependency injection container setup
  app.ts                # Express app setup (middleware, routes, etc.)
  server.ts             # entry point to start the server
/docs                   # documentation (ADRs, HLD, FDDs, PRD, epics, roadmap, etc.)
```

## Tech Stack

- **Runtime:** Node.js (ES Modules — `"type": "module"`)
- **Language:** TypeScript 6 (strict mode, target ES2022, module NodeNext)
- **Framework:** Express.js 5
- **Database:** PostgreSQL 17
- **ORM:** TypeORM 0.3.28
- **Test Runner:** Vitest
- **Dev tooling:** tsx (watch mode), Docker Compose
- **Dependency Injection:** InversifyJS

## Commands

```bash
npm run start:dev       # Dev server with hot reload (tsx watch)
npm run build           # Compile TypeScript → dist/
npm run start:prod      # Run compiled output (node dist/server.js)
npm test                # Run tests once (vitest run)
npm run test:watch      # Run tests in watch mode (vitest)
```

### Docker

```bash
docker compose up       # Start API (port 3000) + PostgreSQL (port 5432)
```

PostgreSQL credentials: user=`lms`, password=`lms`, database=`lms`.

The dev container (`Dockerfile.dev`) uses `node:25.6.0-slim` and runs as the `node` user. Source is volume-mounted at `/home/node/app`.

## Hexagonal Layers Per Module

Each module follows this layered pattern:
- **Routes** → **Controller** → **Use Case** → **Repository (port)** → **Database (adapter)**
- Business logic lives in use cases, isolated from infrastructure
- Repositories are defined as ports (interfaces); implementations are adapters

## Key Documentation

- `docs/domain-context.md` — Personas, entities (15+), and 24 business rules (RN-001 to RN-024)
- `docs/hld.md` — High-Level Design: components, data model, API interfaces, ADRs, risks
- `docs/roadmap.md` — 3-phase release plan
- `docs/prd/` — Product requirements (functional and non-functional)
- `docs/epicos/` — Epic breakdowns (8 epics with user stories and acceptance criteria)

## TypeScript Configuration

- Target: ES2022, Module: NodeNext
- Strict mode with `noUncheckedIndexedAccess`, `verbatimModuleSyntax`, `erasableSyntaxOnly`
- Use `.js` extensions in imports (ESM requirement), e.g., `import app from './app.js'`

## Testing

- **Every implementation must include tests** — no code is considered done without its corresponding tests
- **Testing guide**: the `testing-guide-dev-workflow-test` skill has the complete guide — what to test, at which layer, setup patterns, and anti-patterns per artifact type

## REST Conventions

This is a RESTful API. All endpoints must follow standard REST conventions — correct HTTP methods, proper status codes, plural resource nouns, and consistent URL structure. Details are enforced via rules on controller files.

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
