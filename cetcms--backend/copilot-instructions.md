## backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **CETCMS**, a NestJS-based CMS backend with the following key characteristics:
- **Runtime**: Bun (package.json uses bun commands)
- **Database**: PostgreSQL with Prisma ORM
- **API**: GraphQL (primary) + REST
- **Authentication**: JWT-based with passport-jwt
- **Cache**: Redis + in-memory (Keyv-based)
- **I18n**: i18next with filesystem backend

## Build and Development Commands

```bash
# Development (watch mode)
bun run dev

# Production build
bun run build

# Production run
bun run prod

# Lint and format
bun run lint
bun run format

# Tests
bun run test              # Unit tests
bun run test:watch        # Watch mode
bun run test:e2e          # E2E tests
bun run test:cov          # Coverage

# Database
bun run db:migrate        # Run migrations
bun run db:push           # Push schema changes
bun run db:studio         # Open Prisma Studio
bun run db:seed           # Seed database
bun run generate          # Generate Prisma client

# Scripts
bun run script:scan-permissions          # Scan and generate permissions
bun run script:push-permissions-to-role  # Push permissions to roles
bun run script:gen-i18n-translations     # Generate i18n translations
bun run script:seed-mock                 # Seed mock data
```

## Architecture Overview

### Module Structure

The project follows NestJS modular architecture:

```
src/
├── app.module.ts          # Root module
├── main.ts                # Entry point
├── auth/                  # Authentication module (JWT, guards, decorators)
├── cache/                 # Global cache module (Redis + memory)
├── common/                # Shared utilities, filters, interceptors, tools
├── config/                # Configuration service and definitions
├── contracts/             # Constants and shared contracts
├── database/              # Database module (Prisma)
├── generated/             # Auto-generated from Prisma schema
│   ├── dto/              # Prisma-generator-nestjs-dto output
│   └── graphql/          # prisma-nestjs-graphql output
├── i18n/                  # Internationalization
│   └── translations/     # JSON translation files
├── modules/               # Business modules
│   ├── admin/
│   ├── company/
│   ├── media/
│   ├── member/
│   ├── notification/
│   └── website/
├── providers/             # External service providers
│   ├── site/             # Site crawler
│   └── strapi/           # Strapi CMS integration
└── repositories/          # Data access layer (extends Prisma)
```

### Code Generation

Prisma schema drives code generation:
- **DTOs**: `@brakebein/prisma-generator-nestjs-dto` generates create/update/entity DTOs
- **GraphQL**: `prisma-nestjs-graphql` generates GraphQL types and inputs
- **Zod**: `prisma-zod-generator` generates Zod schemas

After modifying `prisma/schema.prisma`, run `bun run generate`.

### Authentication System

- **Guard**: `JwtAuthGuard` validates JWT tokens
- **Decorator**: `@CurrentAuth()` extracts auth from request context
- **Permission Guard**: `PermissionGuard` checks CASL abilities
- **Roles**: AdminRole (global) and CompanyRole (per-company)

### Repository Pattern

Repositories extend Prisma client with custom methods:
```typescript
// Example: src/repositories/admin/admin.repository.ts
@Injectable()
export class AdminRepository {
  constructor(private readonly prisma: PrismaService) {}
  // Custom methods here
}
```

GraphQL field extensions are in `src/repositories/*/extends.ts` files.

### Key Patterns

1. **Guards**: Apply `JwtAuthGuard` + `PermissionGuard` to protected routes
2. **Decorators**: Use `@CurrentAuth()` to get authenticated user info
3. **I18n**: Service auto-detects language from headers (Accept-Language, X-Language, etc.)
4. **File Upload**: Supports GraphQL multipart uploads via `graphql-upload-ts`
5. **Logging**: Winston with daily rotation in `logs/` directory

### Important Conventions

- **Imports**: Use `src/` path alias for internal imports (configured in tsconfig.json)
- **ESM**: Module system is `nodenext` with ESM output
- **Linting**: ESLint with TypeScript, Prettier integration; single quotes, trailing commas
- **Ignored paths**: `src/generated/**` is excluded from linting

### Environment Variables

Key variables (see `.env.development`):
- `DATABASE_URL` - PostgreSQL connection
- `JWT_SECRET` / `JWT_EXPIRES_IN` - JWT configuration
- Redis config via `ConfigService.getCacheConfig()`

### Testing

- **Unit**: Jest with ts-jest, tests in `*.spec.ts` files
- **E2E**: Separate config in `test/jest-e2e.json`
- **ESM support**: Enabled for both unit and e2e tests

---
> Source: [cetcms/backend](https://github.com/cetcms/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
