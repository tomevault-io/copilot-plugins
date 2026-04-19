## budjet-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm run build          # Build (nest build)
pnpm run start:dev      # Dev server with watch mode
pnpm run start:prod     # Production (node dist/main)
pnpm run lint           # ESLint with auto-fix
pnpm run format         # Prettier formatting
pnpm test               # Run all tests (jest)
pnpm test -- --testPathPattern=<pattern>  # Run a single test file
pnpm run test:e2e       # E2E tests (jest --config ./test/jest-e2e.json)
npx prisma generate     # Regenerate Prisma client (output: src/generated/prisma)
npx prisma migrate dev  # Run database migrations
```

**Note:** ESLint is configured to lint `src/generated/` which produces many errors. When checking lint, use:
```bash
npx eslint "src/**/*.ts" --ignore-pattern "src/generated/**"
```

## Architecture

NestJS 11 API with Prisma ORM on PostgreSQL. Global prefix: `api/v1`. Swagger docs at `api/docs`.

### Modules

- **AuthModule** — JWT auth with access + refresh tokens. Tokens set as httpOnly cookies. Passwords hashed with bcrypt. `JwtAuthGuard` extracts Bearer token from Authorization header.
- **ProductsModule** — Product CRUD. Deletion blocked if product is referenced in transactions.
- **TransactionsModule** — Transaction CRUD with nested transaction items. Duplicate product entries are merged (quantities summed) on creation.
- **UsersModule** — Read-only user endpoints.
- **PrismaModule** — Global database access. Import this module rather than providing `PrismaService` directly.

### Validation

Mixed validation approach:
- **Zod + nestjs-zod**: Auth DTOs and Transaction schemas. `ZodValidationPipe` is registered globally in `main.ts`.
- **class-validator + class-transformer**: Product DTOs.

### Configuration

Environment config is in `src/config/index.ts` (plain object, not NestJS ConfigService). `@nestjs/config` ConfigModule is also registered globally but the codebase primarily uses the custom config object via `import { config } from 'src/config'`.

### Key Patterns

- Services use NestJS `Logger` (not console.log).
- User passwords and refresh tokens are stripped before returning from API responses.
- Auth endpoints (`/sign_up`, `/sign_in`) use snake_case routes.
- Prisma client is generated to `src/generated/prisma/` (configured in `nest-cli.json` to copy assets to dist).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sovsianykov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
