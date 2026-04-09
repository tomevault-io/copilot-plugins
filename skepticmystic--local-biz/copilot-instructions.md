## local-biz

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SvelteKit-based application starter template with TypeScript, TailwindCSS, Better-Auth for authentication, Drizzle ORM for database management, and Redis for caching. Uses Svelte 5 with experimental async components and remote functions. Deployed on Vercel.

**Tech Stack**: Node 22, pnpm 10.23.0, Svelte 5, SvelteKit, TypeScript, Drizzle ORM, PostgreSQL (Neon), Redis (Upstash), Better-Auth, Effect (service DI), Resend (email), Pino (logging)

## Development Commands

### Package Manager

This project uses **pnpm** (v10.23.0) as the package manager. Always use `pnpm` commands instead of `npm`.

### Core Commands

- `pnpm install` - Install dependencies
- `pnpm dev` - Start development server (port 5173)
- `pnpm build` - Build for production
- `pnpm preview` - Preview production build

### Type Checking & Linting

- `pnpm check` - Run svelte-check for type checking
- `pnpm check:watch` - Run svelte-check in watch mode
- `pnpm lint` - Run oxlint (type-aware), eslint, and prettier checks
- `pnpm format` - Format code with prettier

### Database Commands

Database commands use a custom script wrapper (`scripts/drizzle/kit.script.ts`) that invokes drizzle-kit:

- `pnpm db:push` - Push schema changes to database (development)
- `pnpm db:generate` - Generate migrations (production env)
- `pnpm db:check` - Check migration status (production env)
- `pnpm db:migrate` - Apply migrations (used in Vercel build)
- `pnpm db:studio` - Open Drizzle Studio

### Other Tools

- `pnpm knip` - Find unused files, dependencies, and exports

## Architecture

### Directory Structure

- `src/lib/` - Core application logic
  - `auth/` - Authentication configuration and permissions (Better-Auth setup)
  - `clients/` - Client-side API wrappers
  - `components/` - UI components organized by type (auth, form, ui, daisyui)
  - `const/` - Application constants and configuration
  - `external/` - External service integrations (e.g., Discord)
  - `interfaces/` - TypeScript interfaces
  - `remote/` - SvelteKit remote functions (uses experimental feature)
  - `schema/` - Zod validation schemas
  - `server/` - Server-only code
    - `db/` - Database configuration and schemas
  - `services/` - Business logic services (email, config)
  - `stores/` - Svelte stores for state management
  - `utils/` - Utility functions
- `src/routes/` - SvelteKit file-based routing
- `drizzle/` - SQL migrations and metadata
- `scripts/` - Build and utility scripts
- `static/` - Static assets

### Authentication Architecture

- **Better-Auth** integration with custom configuration in `src/lib/auth.ts`
- Uses **Effect** library for dependency injection (email service)
- Split between client (`auth-client.ts`) and server (`auth/server.ts`)
- Database session storage disabled in favor of cookie caching
- Custom session fields for organization membership (`member_id`, `member_role`)
- Automatic organization creation on first login via database hook
- Supports multiple auth providers:
  - Email/Password with verification
  - Google OAuth
  - Generic OAuth (Pocket ID)
  - Passkeys
- **Permissions** managed through `src/lib/auth/permissions.ts` with AccessControl

### Database Architecture

- **Drizzle ORM** with PostgreSQL (Neon)
- Schema files use `*.models.ts` naming convention
- All tables use UUID primary keys (custom ID generation, not BetterAuth's nanoid)
- Database schemas in `src/lib/server/db/schema/`:
  - `auth.models.ts` - User, Session, Account, Organization, Member, Invitation, Passkey, Verification tables
  - `task.models.ts` - Application-specific tables
  - `index.schema.ts` - Shared schema utilities (ID generation, timestamps)
- Uses `snake_case` for database columns (configured in drizzle.config.ts)
- Redis configured as secondary storage for Better-Auth (rate limiting, caching)

### SvelteKit Configuration

- **Experimental features enabled**:
  - `remoteFunctions: true` - Server functions callable from client (see Remote Functions pattern below)
  - `async: true` - Async components in Svelte 5
- Vercel adapter for deployment
- Build command includes database migration: `vite build && pnpm db migrate`

### Application Layer Architecture

The application follows a three-layer architecture pattern:

#### 1. Remote Layer (`src/lib/remote/*.remote.ts`)

Remote functions use SvelteKit's experimental feature to call server code from client:

- Use `form()` from `$app/server` to create type-safe forms
- Use `query()` from `$app/server` to create type-safe queries
- Use `command()` from `$app/server` to create type-safe commands
- First argument: Zod schema for validation
- Second argument: async handler function with validated input
- **Responsibilities**:
  - Handle authentication/authorization (e.g., `get_session()`)
  - Validate input (via Zod schemas)
  - Call Service or Repo layer
  - Handle Result types returned from lower layers
  - For queries that return `App.Result`, use SvelteKit's `error()` helper to throw HTTP errors
- `form` Handler receives `issue` parameter for field-specific validation errors
- Use `invalid(issue.fieldName(message))` to return field-specific errors
- Use `redirect()` to navigate after successful operations
- Example: `src/lib/remote/auth/auth.remote.ts`, `src/lib/remote/business/business.remote.ts`

#### 2. Service Layer (`src/lib/services/*.service.ts`)

Business logic layer that orchestrates operations:

- **Responsibilities**:
  - Add business logic (e.g., slug generation, data transformation)
  - Orchestrate multiple repo operations
  - Handle cross-cutting concerns
  - Return `App.Result` types
- **When to use**:
  - Operations need data transformation (e.g., `BusinessService.create` adds slug)
  - Operations involve multiple database calls
  - Complex business rules need to be enforced
- **When to skip**:
  - Simple CRUD operations with no transformation
  - Read queries with just filtering (can go directly from Remote to Repo)
- Example: `src/lib/services/business/business.service.ts` (adds slug generation)
- **Email Service** (`src/lib/services/email.service.ts`):
  - Uses **Effect** library for dependency injection
  - Define service interface using `Context.Tag`
  - Implement `EmailLive` (Resend) and `EmailTest` (console log) versions
  - Inject at runtime: `Effect.provideService(EmailService, dev ? EmailTest : EmailLive)`
  - Used in auth configuration for verification emails, password resets, org invites

#### 3. Repository Layer (`src/lib/repos/*.repo.ts`)

Data access layer for database operations:

- **Responsibilities**:
  - Raw database operations (queries, inserts, updates, deletes)
  - Always return `App.Result` types with proper error handling
  - Catch `DrizzleQueryError` and unknown errors
  - Log errors with context: `Log.error(error, "RepoName.method.error type")`
  - Capture exceptions with Sentry: `captureException(error)`
  - Return standardized errors using `E.INTERNAL_SERVER_ERROR` or `E.NOT_FOUND` from `$lib/const/error/error.const`
- **Error handling pattern**:
  ```typescript
  try {
    const result = await db.operation();
    return result.suc(data);
  } catch (error) {
    if (error instanceof DrizzleQueryError) {
      Log.error(
        { message: error.message },
        "Repo.method.error DrizzleQueryError",
      );
      captureException(error);
      return result.err(E.INTERNAL_SERVER_ERROR);
    } else {
      Log.error(error, "Repo.method.error unknown");
      captureException(error);
      return result.err(E.INTERNAL_SERVER_ERROR);
    }
  }
  ```
- **Naming conventions**:
  - Generic operations: `create`, `update`, `delete`
  - Specific queries: `get_all_public`, `get_all_by_user`, `set_admin_approved`
- Example: `src/lib/repos/business.repo.ts`

### State Management

- Svelte 5 runes for reactive state
- Stores in `src/lib/stores/` for shared state (organizations, session)
- Better-Auth client provides session management

### Logging

- Custom logging utility using **Pino** (`src/lib/utils/logger.util`)
- Configured with pretty-printing in development
- Use `Log.info()`, `Log.error()`, `Log.debug()`, etc. throughout codebase
- Log level controlled by `LOG_LEVEL` environment variable

## Linting Configuration

### Oxlint

Primary linter with type-aware checking configured in `.oxlintrc.json`:

- TypeScript and Unicorn plugins enabled
- Correctness category disabled (relies on TypeScript compiler)
- Strict unused variable rules with `_` prefix for intentionally unused vars
- Special rules for Svelte files (no-inner-declarations, no-self-assign disabled)

### ESLint

Secondary linter in `eslint.config.js` - runs after oxlint in the lint pipeline.

## Environment Setup

1. Ensure **Node 22** is installed (required by package.json engines)
2. Install **pnpm 10.23.0** (specified in package.json packageManager)
3. Create `.env` file based on `.env.example`
4. Set up PostgreSQL database (Neon recommended) with development branch
5. Add `DATABASE_URL` to `.env`
6. Optional: Configure Redis with `REDIS_URL` (for Better-Auth rate limiting and caching)
7. Run `pnpm install` to install dependencies
8. Run `pnpm db:push` to create tables
9. Configure auth provider credentials as needed (Google, Pocket ID)
10. Configure email service (Resend) with `RESEND_API_KEY` and `EMAIL_FROM`

## Deployment (Vercel)

1. Connect repository to Vercel
2. Set environment variables in Vercel dashboard (see `.env.example`)
3. Create `.env.production` for production database URL (used by `pnpm db:generate` and `pnpm db:check`)
4. Edit build command to: `vite build && pnpm db:migrate`
5. Deploy

## Key Patterns and Conventions

### Code Organization

- **Naming conventions**:
  - Database schema files: `*.models.ts`
  - Remote functions: `*.remote.ts`
  - Services: `*.service.ts`
  - Utilities: `*.util.ts`
- **Import aliases**: Use `$lib`, `$app`, `$env` SvelteKit aliases
- **TypeScript namespaces**: Preferred for organizing related types (e.g., `IAuth.ProviderId`)

### Database Patterns

- All tables use UUID primary keys via `Schema.id()` from `index.schema.ts`
- Timestamps use `Schema.timestamps` helper (createdAt, updatedAt)
- Database columns in `snake_case`, TypeScript in `camelCase` (Drizzle handles conversion)
- Never use BetterAuth's nanoid generation; custom UUID generation is configured

### Error Handling

- **Result Type Pattern**: Use `App.Result<T>` type for operations that can fail
  - Success: `result.suc(data)` returns `{ ok: true, data: T }`
  - Error: `result.err(error)` returns `{ ok: false, error: E }`
  - Check results with `if (!res.ok)` then access `res.error` or `res.data`
- **Standardized Errors**: Use constants from `$lib/const/error/error.const.ts`:
  - `E.INTERNAL_SERVER_ERROR` - Generic server errors
  - `E.NOT_FOUND` - Resource not found
  - Include `status` and `message` properties
- **Logging**: Use Pino logger with context
  - `Log.error(error, "context_identifier")`
  - `Log.info()`, `Log.debug()`, `Log.warn()` for other levels
- **Exception Tracking**: Always `captureException(error)` for Sentry in repo layer
- **Better-Auth Errors**: API errors are instances of `APIError` with `body.code` for error types
- **Custom Error Codes**: Defined in `$lib/auth-client` as `$ERROR_CODES`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SkepticMystic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SkepticMystic)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
