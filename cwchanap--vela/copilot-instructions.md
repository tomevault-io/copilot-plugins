## vela

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a **Turborepo monorepo** containing:

- `apps/vela` - Main Vela Japanese learning app (Quasar/Vue.js)
- `apps/vela-api` - API backend (Hono framework, runs on port 9005)
- `apps/vela-ext` - Browser extension for saving Japanese sentences (WXT/Vue.js)
- `packages/cdk` - AWS CDK infrastructure (5 stacks: Auth, Database, Storage, Api, StaticWeb)
- `packages/common` - Shared TanStack Query config and query key factories

## Commands

### Monorepo (from root)

```bash
bun run dev          # Start all dev servers in parallel
bun run build        # Build all packages
bun run test         # Run all unit tests
bun run lint         # Lint all packages
bun run lint:fix     # Lint and fix
bun run format       # Format all packages
bun run clean        # Clean build artifacts
```

### Vela App (from apps/vela/)

```bash
bun run dev                                      # Quasar dev server (port 9000)
bun run test:unit                                # Run Vitest unit tests
bun run test:unit -- --coverage                  # Run with coverage (v8 provider)
bun vitest run src/components/auth/AuthForm.test.ts  # Run single test file
bun run test:e2e                                 # Run Playwright e2e tests
bun run test:e2e tests/auth.spec.ts              # Run single e2e test file
bun run test:headed                              # Playwright in headed mode
bun run test:ui                                  # Playwright UI mode
bun run build                                    # Production build
```

### Vela API (from apps/vela-api/)

```bash
bun run dev           # Bun watch mode (NODE_ENV=development bun --watch src/index.ts)
bun run test:unit     # Run tests with Bun's built-in test runner
bun run test:coverage # Run with coverage
bun test test/routes/profiles.test.ts  # Run single test file
bun run build         # Bundle for Lambda deployment
```

### Vela Extension (from apps/vela-ext/)

```bash
bun run dev           # WXT dev server for Chrome
bun run dev:firefox   # WXT dev server for Firefox
bun run build         # Build for Chrome
bun run zip           # Create distribution zip
bun run compile       # Type-check via vue-tsc
```

### CDK (from packages/cdk/)

```bash
bun cdk:synth    # Synthesize CloudFormation template
bun cdk:diff     # Diff deployed vs local
bun cdk:deploy   # Deploy to AWS
bun lambda:build # Build Lambda bundle via esbuild
```

## Architecture

### Key cross-package pattern

`@vela/common` provides shared TanStack Query config (`packages/common/src/config.ts`) and query key factories (`packages/common/src/keys.ts`). Both `apps/vela` and `apps/vela-ext` depend on it. If those packages appear broken after pulling, build `@vela/common` first:

```bash
bun run build --filter=@vela/common
```

### Vela App

- **Boot**: `src/boot/query.ts` initializes TanStack Query (stale: 5m, gc: 10m); `src/boot/main.ts` is minimal
- **State**: Pinia stores in `src/stores/` — `auth.ts` manages the full Cognito session lifecycle
- **Services**: `src/services/` contains business logic; components call services, not the API directly
- **Config**: `src/config/index.ts` validates all env vars at startup — misconfigured env fails fast
- **Routing**: Protected routes use `requiresAuth: true` meta; guest-only use `requiresGuest: true`

### Vela API

Key files:

- `src/index.ts` — Hono app entry point; handles both Lambda and local Bun runtime
- `src/dynamodb.ts` — DynamoDB client and typed operations
- `src/dsql.ts` — Aurora DSQL client (PostgreSQL via `@aws/aurora-dsql-node-postgres-connector`)
- `src/middleware/auth.ts` — JWT verification via `aws-jwt-verify`
- `src/validation.ts` — Zod schemas for request validation

Routes: `auth`, `profiles`, `games`, `progress`, `chat-history`, `llm-chat`, `my-dictionaries`, `tts`, `srs`, `dsql-health`

**Dev vs production**: In development, env vars are loaded from `apps/vela-api/.env` or root `.env`. The Hono app detects Bun runtime automatically and skips the Lambda handler wrapper.

### Development proxy

In development, `quasar.config.ts` proxies all `/api/*` requests from port 9000 → port 9005. No CORS configuration is needed locally. In production, CloudFront routes `/api/*` → API Gateway and `/*` → S3.

### Database

- **DynamoDB**: 8 tables for all current application data (profiles, vocabulary, sentences, game sessions, daily progress, chat history, saved sentences, TTS settings)
- **Aurora DSQL**: PostgreSQL-compatible serverless cluster in private VPC subnet — the target relational DB for future SQL workloads (migration in progress)

### Infrastructure stacks (packages/cdk/)

```
AuthStack → (no deps)          # Cognito User Pool
DatabaseStack → (no deps)      # DynamoDB + Aurora DSQL + VPC
StorageStack → (no deps)       # S3 for TTS audio
ApiStack → [auth, db, storage] # Lambda + API Gateway
StaticWebStack → [all above]   # CloudFront + S3 static hosting
```

## Testing

- **E2E tests** require `TEST_EMAIL` and `TEST_PASSWORD` env vars (see `.env.example`)
- **Vitest** uses jsdom environment with globals enabled; setup file at `src/test/setup.ts`
- **Vitest aliases**: `@vela/common` is aliased directly to the source (`packages/common/src/index.ts`) — no build step needed for unit tests
- **API tests** use Bun's built-in test runner (no Vitest)

## Environment Variables

- **Vela App**: `VITE_*` prefix, Cognito config required — validated at startup via `src/config/index.ts`
- **Vela API**: Standard `process.env`; dev mode falls back between `apps/vela-api/.env` and root `.env`
- **Vela Extension**: WXT built-in env handling

## Code Quality

ESLint uses flat config format (v9+). Pre-commit hooks (Husky + lint-staged) run lint and format automatically. TypeScript strict mode is on across all packages with Vue SFC type checking via `vite-plugin-checker` in the Vela app.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cwchanap) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
