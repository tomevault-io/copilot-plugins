## arche-cms

> Arche CMS — a file-based, open-source, developer-first headless CMS (Strapi + Payload inspired). Schema is stored in source files, not in the database. Everything is designed for extensibility, plugins, type safety, and excellent developer experience.

# AGENTS.md — Arche CMS

## Project

Arche CMS — a file-based, open-source, developer-first headless CMS (Strapi + Payload inspired). Schema is stored in source files, not in the database. Everything is designed for extensibility, plugins, type safety, and excellent developer experience.

## Principles

- Domain Driven Design
- SOLID
- Clean Architecture
- Modular Monolith
- Feature-first organization
- Dependency Injection where appropriate
- Event Driven internals
- Type-safe APIs
- Minimal runtime magic, no hidden behavior
- Never sacrifice architecture for speed

Priority order: 1) Developer Experience, 2) Type Safety, 3) Performance, 4) Extensibility, 5) Clean Architecture, 6) Plugin System, 7) Testability

## Technology Stack

### Frontend

- React 19, TypeScript, Vite, Tailwind CSS v4, shadcn/ui
- TanStack Router, TanStack Query

### Backend

- Node.js, TypeScript, Fastify

### Database

- Drizzle ORM with adapter pattern
- SQLite (initial), PostgreSQL (initial), MySQL, Turso, Cloudflare D1, MongoDB (future)

### Validation

- Zod

### Auth

- JWT (initial), OAuth / Magic Links / Passkeys (future)

### Storage

- Local, S3, Cloudflare R2

### Package Manager

- pnpm

### Repository

- Turborepo

## Monorepo Structure

```
arche-cms/
├── apps/
│   ├── docs/           # Documentation site
│   └── playground/     # Dev playground
├── packages/
│   ├── admin-ui/       # Admin panel UI (React 19, Vite, TanStack Router, shadcn/ui)
│   │   ├── src/        # Components, routes, hooks, API client
│   │   └── dist/       # Bundled admin panel build
│   ├── cms/            # CMS package (CLI, server, API)
│   │   ├── src/        # CLI + server logic
│   │   ├── dist/       # Compiled output
│   │   └── bin/        # CLI binary
│   ├── core/           # Core framework, DI, event bus
│   ├── schema/         # Schema definition API (defineCollection, field helpers)
│   ├── database/       # Database adapter layer (Drizzle ORM)
│   ├── auth/           # Authentication (JWT, OAuth, etc.)
│   ├── permissions/    # RBAC / permissions engine
│   ├── storage/        # File storage adapters (local, S3, R2)
│   ├── rest-api/       # REST API generator
│   ├── graphql/        # GraphQL schema generator
│   ├── plugins/        # Plugin system + official plugins
│   ├── cli/            # CLI tools (cms dev, build, generate, etc.)
│   ├── generators/     # Code generation pipeline (types, routes, migrations, SDK)
│   ├── sdk/            # TypeScript client SDK
│   └── types/          # Shared TypeScript types
```

## Schema Example

```ts
export default defineCollection({
  slug: "posts",
  labels: { singular: "Post", plural: "Posts" },
  fields: [
    text("title"),
    slug("slug"),
    richText("content"),
    media("featuredImage"),
    relation("author", { to: "users" }),
    select("status", { options: ["draft", "published"] }),
  ],
});
```

Files live in `cms/collections/*.ts`, `cms/globals/*.ts`, `cms/components/*.ts`. The CMS auto-loads schemas, validates them, generates types, creates DB tables/migrations, builds the Admin UI, exposes REST + GraphQL APIs, validates requests, and handles permissions.

## Conventions

### Code style

- TypeScript strict mode
- Functional components with hooks (no class components)
- Use `const` over `let`/`var`
- Named exports over default exports
- No commented-out code or `console.log` in committed code

### Imports

Order: 1) Node built-ins, 2) third-party packages, 3) project aliases (`@/`), 4) relative imports. No blank lines between groups.

### File structure

- Packages follow feature-first organization under `packages/<name>/`
- Each package has `src/`, `test/`, `README.md`
- Components in `src/components/` (one file per component, PascalCase)
- Hooks in `src/hooks/` (camelCase, `use` prefix)
- Utils in `src/lib/` (camelCase)
- Types in `src/types/` (PascalCase)
- Routes in `src/routes/` (TanStack Router file conventions)

### Testing

- Vitest
- Tests next to source files: `*.test.ts` / `*.spec.ts`
- E2E tests with Playwright in `packages/cms/src/admin/e2e/`

### Commits

- Conventional commit format: `type(scope): message` (e.g. `feat(schema): add richText field type`)
- No `--no-verify` or force push
- Never commit secrets or `.env` files

### Pull requests

- Title matches first line of commit (if single commit) or describes the change
- Include summary of what and why

## Development Workflow

For every feature:

1. **Design** — architecture and approach
2. **Architecture** — how it fits into the modular monolith
3. **Types** — define schemas and interfaces first
4. **Tests** — write tests before implementation
5. **Implementation** — write production code
6. **Documentation** — README, API docs, architecture notes

Do not skip steps. Do not implement features until the architecture has been reviewed.

## Commands

- `pnpm dev` — start dev servers
- `pnpm build` — build all packages
- `pnpm lint` — lint all packages
- `pnpm typecheck` — TypeScript type checking
- `pnpm test` — run all tests
- `pnpm test:watch` — test watch mode
- `pnpm format` — format with Prettier
- `pnpm knip` — detect unused exports, dependencies, and types
- `pnpm knip:fix` — auto-fix knip findings
- `cms dev` — start CMS dev mode (CLI)
- `cms build` — build CMS for production
- `cms generate` — run code generation
- `cms migrate` — run database migrations
- `cms typegen` — generate TypeScript types from schemas
- `cms plugin create` — scaffold a new plugin
- `cms collection create` — scaffold a new collection
- `cms doctor` — check project health
- `cms lint` — lint schema definitions

## Field Types

text, textarea, number, boolean, date, datetime, email, password, url, json, richText, markdown, code, color, media, upload, select, multiSelect, radio, checkbox, relation, component, dynamicZone, array, object, tabs, group, repeater

## Content Features

Draft/publish, version history, autosave, revisions, scheduled publishing, localization, slug generation, SEO fields, preview URLs, soft delete

## Plugin System

Everything is pluggable. Plugins register themselves automatically. Examples: Search, SEO, Comments, Audit Log, Analytics, Email, Payments, Custom Fields, AI, Workflows, Webhooks.

## API Generation

Auto-generated: REST, GraphQL, OpenAPI, TypeScript SDK, Zod validation schemas.

## Code Generation Pipeline

Auto-generates: TypeScript types, API routes, Zod validation, DB schema, DB migrations, permissions, GraphQL schema, OpenAPI spec, SDK, admin forms.

## Documentation

Every package contains: README, architecture notes, API documentation.

---
> Source: [Arche-CMS/arche-cms](https://github.com/Arche-CMS/arche-cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
