## evadiagnostica

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Dev
yarn run start:dev        # watch mode
yarn run build            # production build
yarn run start:prod       # run dist/main

# Test
yarn run test             # all unit tests (rootDir: src, *.spec.ts)
yarn run test:watch
yarn run test:cov
yarn run test:e2e         # uses test/jest-e2e.json

# Lint / format
yarn run lint
yarn run format

# Database
yarn typeorm migration:generate -- infrastructure/database/migrations/<Name> -d infrastructure/database/data-source.ts
yarn typeorm migration:run -d infrastructure/database/data-source.ts
yarn typeorm migration:revert -d infrastructure/database/data-source.ts

# Seeders
yarn db:seed
```

Podman services (Postgres + Redis):
```bash
podman compose -f compose.yml up -d
```

`synchronize: true` is active when `APP_ENV !== 'production'` — schema auto-syncs in dev.

## Path Aliases

```
@modules/*        → src/*
@database/*       → infrastructure/database/*
@config/*         → infrastructure/config/*
@Helpers/*        → infrastructure/helpers/*
@Exceptions/*     → infrastructure/exceptions/*
@infrastructure/* → infrastructure/*
@Common/*         → src/common/*
```

## Architecture

NestJS monolith. Domain-Driven structure per module: `domain/`, `application/`, `infrastructure/`.

```
src/
  auth/
    domain/services/       # interfaces (BlackListService, JwtService)
    application/           # use cases: LoginUserCase, RefreshTokenCase
    application/services/  # AuthTokenService (token generation + session persistence)
    infrastructure/
      controllers/         # AuthController
      guards/              # AccessTokenGuard, RefreshTokenGuard (Passport strategies)
      services/            # BlackListServices (Redis), TokenService (JWT)
      decorators/          # @SessionMeta — extracts ip + user-agent from request
  users/
    domain/entities/       # User, UserSession (TypeORM)
    domain/repositories/   # UserRepository, UserSessionRepository
    domain/services/       # UserCacheService interface
    infrastructure/services/
      user-cache.services  # UserCacheServiceImpl (Redis via CacheModule)
      session-cleanup.task # @Cron every 6h — deletes expired UserSession rows
  authorization/
    domain/entities/       # Role, Permission (ManyToMany on User)
    domain/Enums/          # permissions.enum
  common/
    results.ts             # Result<T,E> discriminated union — used by all use cases

infrastructure/
  config/         # app, jwt, redis, hashing, timezone configs (loaded by ConfigModule)
  database/       # DatabaseModule, AppDataSource, SnakeNamingStrategy, seeders
  exceptions/     # GlobalExceptionFilter
  helpers/        # Hash (argon2/bcrypt wrapper)
```

## Key Patterns

**Result type** — use cases return `Result<T, E>`. Controllers check `result.ok` and throw HTTP exceptions on failure. Never throw from use cases.

**Token auth flow:**
1. Login → `LoginUserCase` → `AuthTokenService.generateAndPersistTokens` → saves two `UserSession` rows (access + refresh JTI) → returns both tokens.
2. Refresh → `RefreshTokenGuard` (passport-jwt strategy `jwt-refresh`) validates token + blacklist, injects payload → `RefreshTokenCase` → reads user from Redis cache or DB → calls `AuthTokenService.refreshAuthTokens`. Refresh token only rotates when remaining TTL < `tokenRotationThreshold`.
3. Access guard validates token type = `'access'`, checks both blacklist keys: `black:version:<userId>:<version>` and `black:jti:<jti>`.

**Blacklist** — Redis-backed via `@nestjs/cache-manager` + Keyv/Redis. Two key prefixes: `black:version:` (invalidates all tokens at a version) and `black:jti:` (invalidates a single token).

**User cache** — `UserCacheService` wraps CacheModule to cache user+permissions in Redis. `RefreshTokenCase` reads from cache first, falls back to DB and re-populates.

**Platform header** — `x-client-platform: mobile` receives refresh token in JSON body; web receives it as `httpOnly` cookie on `/api/auth`.

**Permissions** — unified permissions = role permissions ∪ direct user permissions. Codes (numbers) go into JWT payload; names go into login response.

**Session cleanup** — `SessionCleanupTask` cron (default every 6h, overridable via `SESSION_CLEANUP_CRON` env) deletes expired `UserSession` rows.

## Naming Conventions

- Use cases: `<Action><Entity>Case` (e.g. `LoginUserCase`)
- Services suffix: `.services.ts`, `.service.ts`, `.task.ts`
- Guards: `<TokenType>Guard` + `<TokenType>Strategy`
- DB entities use snake_case columns via `SnakeNamingStrategy`

---
> Source: [Piero19-cyber/evadiagnostica](https://github.com/Piero19-cyber/evadiagnostica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
