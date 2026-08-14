## lumen-here-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a NestJS GraphQL API (lumen_here_api_v2) built with TypeScript, Prisma ORM, and MySQL database. The application follows Domain-Driven Design (DDD) principles with Clean Architecture patterns.

# rule

- 민감한 값은 마스킹 해.
- 답변 요약은 불릿 3~5개.
- 명령 실행이 필요하면 먼저 계획→명령→결과 순으로 한국어로 보여줘.
- **any 타입 사용 금지**: TypeScript에서 any 타입 사용을 최대한 지양할 것. 명확한 타입 정의나 제네릭 사용을 우선함.

## Development Commands

**Package Manager**: Uses `pnpm` (not npm/yarn)

### Setup

- `pnpm install` - Install dependencies
- Database setup requires `.env` file with `DATABASE_URL` for MySQL connection

### Development

- `pnpm run start:dev` - Start development server with hot reload
- `pnpm run start:debug` - Start with debug mode
- `pnpm run build` - Build for production
- `pnpm run start:prod` - Start production server

### Testing

- `pnpm run test` - Run unit tests
- `pnpm run test:watch` - Run tests in watch mode
- `pnpm run test:integration` - Run integration tests (uses `jest-integration.json` config)
- `pnpm run test:cov` - Run tests with coverage
- `pnpm run test:debug` - Debug tests

### Code Quality

- `pnpm run lint` - ESLint with auto-fix
- `pnpm run format` - Prettier formatting

### Database

- Prisma client is generated to `generated/prisma/`
- Database migrations in `prisma/migrations/`

## Architecture

### DDD Layer Flow Policy

**Layer Dependency Rules** (MUST follow these rules):

```
Presentation → Application → Domain ← Infrastructure
```

**Layer Responsibilities:**
- **Presentation**: Handles external requests (GraphQL resolvers, REST controllers)
- **Application**: Orchestrates use cases, manages transactions, calls domain services
- **Domain**: Core business logic, entities, value objects, domain services (no external dependencies)
- **Infrastructure**: External system integrations, implements domain interfaces (repositories, external APIs)

**Dependency Direction Rules:**
- Inner layers CANNOT depend on outer layers
- Domain layer is the core - NO dependencies on other layers
- Infrastructure implements domain interfaces (Dependency Inversion Principle)
- Application layer coordinates between domain and infrastructure
- Presentation layer only calls application layer

**Forbidden Dependencies:**
- ❌ Domain → Infrastructure (use interfaces instead)
- ❌ Domain → Application  
- ❌ Domain → Presentation
- ❌ Presentation → Infrastructure (go through application)

### Module Structure

Follows DDD layered architecture within each domain:

```
src/
├── main.ts                    # Application bootstrap
├── app.module.ts             # Root module
├── schema.gql                # Generated GraphQL schema
├── common/                   # Shared infrastructure
│   ├── exceptions/           # Custom exception classes
│   ├── filters/             # Global exception filters
│   └── infrastructure/      # Shared services (Prisma)
├── user/                    # User domain
│   ├── application/         # Application services & DTOs
│   ├── domain/              # Domain entities, value objects, interfaces
│   ├── infrastructure/      # Repositories, external services
│   ├── presentation/        # GraphQL resolvers & DTOs
│   └── user.module.ts
└── auth/                    # Authentication domain
    ├── application/         # Auth services
    ├── domain/              # Token services
    ├── infrastructure/      # JWT implementation
    ├── presentation/        # Auth resolvers
    └── auth.module.ts
```

### Key Patterns

**Exception Handling**: Use custom exception classes from `@common/exceptions`:

- `CustomBadRequestException` - 400 Bad Request errors
- `CustomNotFoundException` - 404 Not Found errors  
- `CustomUnauthorizedException` - 401 Unauthorized errors
- `CustomForbiddenException` - 403 Forbidden errors
- `CustomInternalServerException` - 500 Internal Server errors
- All extend `BaseException` with consistent error structure

**Dependency Injection**: Uses interface-based DI with token providers:

- `'IUserRepository'` → `UserRepository`
- `'IPasswordEncryptionService'` → `CryptoPasswordEncryptionService`
- `'ITokenService'` → `JwtTokenService`

**Path Aliases**: Configured in `tsconfig.json`:

- `@/*` → `src/*`
- `@common/*` → `src/common/*`
- `@user/*` → `src/user/*`

**GraphQL**: Apollo Server with:

- Code-first approach (decorators)
- Auto-generated schema at `src/schema.gql`
- Playground enabled at `/graphql`

**Database**: Prisma ORM with MySQL

- Models: `User` (id, email, name, password, salt, timestamps)
- Generated client in `generated/prisma/`

### Application Configuration

- **Port**: 3000
- **GraphQL Endpoint**: `http://localhost:3000/graphql`
- **Global Pipes**: ValidationPipe with whitelist, transform
- **Global Filters**: Custom exception handling
- **CORS**: Enabled
- **JWT**: Configurable via `JWT_SECRET` and `JWT_EXPIRES_IN` env vars

## Testing Structure

- **Unit Tests**: `*.spec.ts` files in each module
- **Integration Tests**: `*.integration.spec.ts` with separate jest config
- **Setup**: `jest-integration.setup.ts` for integration test environment

---
> Source: [instant2002/lumen_here_api](https://github.com/instant2002/lumen_here_api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
