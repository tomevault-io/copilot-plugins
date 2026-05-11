## goalden-auth-service

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start with hot-reload (tsx watch)
npm run build     # Compile TypeScript → dist/
npm start         # Run compiled output
npm test          # Run Jest tests (uses --experimental-vm-modules for ESM)
```

Run a single test file:
```bash
node --experimental-vm-modules node_modules/jest/bin/jest.js src/path/to/file.test.ts
```

## Architecture

This is a standalone Node.js + TypeScript (ESM) microservice. It runs on port `3001` by default and connects to MongoDB (`goalden_auth` database).

### Layers

```
routes/        → Express routers (thin, delegate to services)
services/      → Business logic (AuthService, TokenService)
repositories/  → MongoDB access via Mongoose (UserRepository, RefreshTokenRepository)
models/        → Mongoose schemas (User, RefreshToken, PasswordResetToken)
middleware/    → errorHandler, userAuth (JWT verify), serviceAuth (service JWT verify), validate (Joi)
validators/    → Joi schemas used by validateBody() middleware
config/        → Single config object from env vars
types/         → Shared interfaces + SERVICE_PERMISSIONS registry
```

### Two JWT systems

1. **User JWTs** — issued on login/register, signed with `USER_JWT_SECRET`, carry `{ userId, email }`. Validated by `userAuth.ts` middleware on user-facing routes.
2. **Service JWTs** — issued at `POST /internal/service-token` (authenticated via a pre-shared `serviceId:serviceSecret` pair from `SERVICE_SECRETS`), signed with `SERVICE_JWT_SECRET`, carry `{ serviceId, scope: "internal", permissions[] }`. Validated by `serviceAuth.ts` middleware on all other `/internal/*` routes.

Permissions per service are defined in `src/types/index.ts` → `SERVICE_PERMISSIONS`.

### Refresh token flow

Refresh tokens are opaque random hex strings stored in MongoDB (`RefreshToken` collection) with an `expiresAt` field. On `POST /api/auth/refresh`, the old token is deleted and a new one is issued (rotation). On logout, all tokens for the user are deleted.

### Error handling

All errors should be thrown as `AppError(statusCode, code, message)` from `middleware/errorHandler.ts`. The global `errorHandler` middleware converts them to `{ error, message }` JSON responses. Mongoose `ValidationError` and duplicate-key errors (code 11000) are also handled.

## Environment variables

Copy `.env.example` to `.env`. Required secrets:
- `USER_JWT_SECRET` — signs user access tokens
- `SERVICE_JWT_SECRET` — signs inter-service tokens
- `SERVICE_SECRETS` — comma-separated `serviceId:secret` pairs for services that can request service tokens
- `MONGO_URI` — defaults to `mongodb://localhost:27017/goalden_auth`

## TypeScript configuration

Strict mode is enabled with `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`. The project uses `NodeNext` module resolution — always use `.js` extensions in imports even for `.ts` source files.

---
> Source: [Minitsonga/Goalden-auth-service](https://github.com/Minitsonga/Goalden-auth-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
