## alpadev-ai-monorepo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Repository Layout

Turborepo + pnpm workspaces. Node >= 22, pnpm 10.11+. Workspace globs: `apps/*`, `packages/**/*`.

```
apps/frontend/              Next.js 15 app (workspace name: "next-app-template")
packages/api/               tRPC v11 router + Express webhooks + BullMQ workers
packages/auth/              NextAuth (v4 + v5-beta transition — see trpc.ts note)
packages/db/                Prisma 6 client + MongoDB schema
packages/email/             React Email templates + Resend
packages/validations/       Zod schemas (single source of truth for API + forms)
packages/utils/             Shared utilities, feature flags
packages/web3/              Web3 helpers
packages/configs/           eslint / prettier / tailwind / tsconfig workspaces
.claude/rules/              Domain-specific rules — read before editing a domain
```

## Architecture

### Data flow

`apps/frontend` → tRPC client → `packages/api` (`appRouter` in `packages/api/src/root.ts`) → domain router → service → repository → `@package/db` (Prisma + Mongoose where noted).

### API layering (every domain in `packages/api/src/routers/{domain}/`)

```
{domain}.repository.ts    Prisma/Mongoose queries only — no business logic
{domain}.service.ts       Business rules, orchestration, TRPCError throws
{domain}.router.ts        tRPC procedures — validate input (Zod), delegate to service
index.ts                  Re-exports
```

Do not inline Zod schemas in routers. Import from `@package/validations`.

### tRPC procedures (defined in `packages/api/src/trpc.ts`)

- `publicProcedure` — no auth
- `protectedProcedure` — requires session; blocks un-onboarded users via `featureFlags.onboardingFlow` (except `user.updateUserOnboarding`)
- `adminProcedure` — requires `role === "ADMIN"`
- `chiefProcedure` — `ADMIN` or `CHIEF`
- `loggedProcedure` — fire-and-forget activity logging for non-ADMIN users; sanitizes password/token fields and truncates strings >200 chars

### Domain routers (17 — see `packages/api/src/root.ts`)

`user`, `admin`, `request`, `newsletter`, `transaction`, `bill`, `invoice`, `cloudflare`, `booking`, `calendar`, `chatbot`, `chat`, `prospect`, `infrastructure`, `permission`, `activity`, `document`

### Storage / external services

- **MongoDB** via Prisma (`MONGO_URL`) — must run as replica set `rs0` for transactions
- **Cloudflare R2** via `@aws-sdk/client-s3` (S3-compatible, pre-signed URLs)
- **Twilio** webhooks (WhatsApp) — Express router in `packages/api/src/webhooks/`, runs on port 3003, requires signature verification
- **Resend** for transactional email (React Email templates)
- **Google Calendar** OAuth — bidirectional sync with `booking` domain
- **Genkit AI** (`packages/api/src/config/ai.config.ts`) with Mistral + Google AI + Deepseek plugins; prompts live in `packages/api/src/prompts/`
- **BullMQ + Redis** for background jobs (`packages/api/src/jobs/chat/`, `packages/api/src/jobs/document/`)

### Frontend specifics

- Custom Next.js server (`apps/frontend/server.ts`) — required for live chat WebSockets on `/ws`. `pnpm dev` uses this server, not `next dev`.
- App Router (`apps/frontend/app/`), server components by default; `"use client"` only when needed (hooks, event handlers, R3F)
- Three.js via `@react-three/fiber` + drei; lazy-load scenes with `dynamic(() => import('./Scene'), { ssr: false })`
- tRPC UI docs at `/api/docs` in development only
- `tsconfig` target is `es5` — use `Array.from()` for Set/Map iteration

### API package export boundary

`packages/api/package.json` exposes only `"."` and `"./types"`. New public symbols must be re-exported from `packages/api/src/index.ts`.

## Commands

### Root (from repo root)

```
pnpm dev                     Redis (docker) + turbo dev (full stack)
pnpm dev:no-redis            turbo dev without starting redis
pnpm build                   turbo build (cascades db:generate first)
pnpm typecheck               turbo typecheck
pnpm lint                    turbo lint + manypkg check (monorepo consistency)
pnpm format                  prettier across repo
pnpm db:push                 push Prisma schema to MongoDB (no migrate)
pnpm db:studio               Prisma Studio
```

### Workspace-scoped

```
pnpm --filter @package/api webhooks:dev       Express Twilio webhook server (port 3003)
pnpm --filter @package/api worker:chat        BullMQ chat worker
pnpm --filter @package/api worker:document    BullMQ document worker
pnpm --filter @package/api typecheck          type-check API only
pnpm --filter next-app-template dev           frontend only (custom ws server)
pnpm --filter next-app-template typecheck     tsc --noEmit on frontend
pnpm --filter next-app-template lint          eslint frontend
```

### Database (from `packages/db/`)

```
pnpm db:generate             prisma generate (runs automatically on install)
pnpm db:push                 prisma db push --skip-generate
pnpm db:migrate              prisma migrate dev
pnpm db:deploy               prisma migrate deploy (production)
pnpm db:seed                 run prisma/seed.ts (requires MONGO_URL env + rs0 replica set)
pnpm db:seed:admin           seed admin user only
pnpm db:reset                DESTRUCTIVE — wipes DB
```

### Tests

```
pnpm --filter next-app-template exec playwright test                  all E2E
pnpm --filter next-app-template exec playwright test path/file.spec   single E2E file
pnpm --filter next-app-template exec playwright test -g "pattern"     filter by name
```

E2E specs live in `apps/frontend/e2e/`. No root `test` script is wired — unit tests use Vitest per-package where configured.

### Twilio webhook local setup

Requires ngrok on port 3003. Set `WEBHOOK_BASE_URL` to the ngrok URL in `.env` and register it in Twilio sandbox console. See README.md §WhatsApp Webhook Setup.

## Conventions

### Cross-package rules

Domain-specific rules live in `.claude/rules/`. **Read the relevant rule file before editing that domain:**

- `general.md` — naming, commit format, monorepo boundaries
- `trpc-api.md` — repository/service/router layering, procedure types
- `domain.md` — business workflows, status enums (Request/Booking/Bill/Invoice/Transaction)
- `validators.md` — Zod schema conventions (ObjectId regex, `z.nativeEnum` for Prisma enums)
- `frontend.md` — Next.js 15, R3F cleanup, GSAP/Framer/Lottie usage
- `ai-integration.md` — Genkit prompt/flow patterns, provider selection
- `security.md` — auth levels, secrets, R2 pre-signed URLs, webhook signatures
- `errors.md` — TRPCError code mapping
- `testing.md` — Playwright + Vitest patterns, `createCallerFactory` for router tests
- `email.md` — React Email + Resend
- `webhook-handlers.md` — Express + Twilio signature verification + idempotency
- `infrastructure.md` — Docker, Turborepo env vars, CI/CD

### Imports

- Frontend/API import validators from `@package/validations` — never duplicate schemas
- DB access only via `@package/db` (single Prisma client instance)
- Named exports only (exception: Next.js page components, R3F Canvas children)
- Kebab-case files, PascalCase React components, camelCase functions

### Git

Conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`). Branch naming: `feat/<slug>`, `fix/<slug>`. No direct pushes to `main` — PR required.

### Adding env vars

Turborepo filters env by `globalEnv` in `turbo.json`. New env vars must be added there or builds won't see them.

---

# Agent Directives (v3)

Hooks handle verification mechanically. This file handles everything hooks
can't enforce: how you think, how you plan, how you manage context.

## Planning

- When asked to plan: output only the plan. No code until told to proceed.
- When given a plan: follow it exactly. Flag real problems and wait.
- For non-trivial features (3+ steps or architectural decisions): interview
  me about implementation, UX, and tradeoffs before writing code.
- Never attempt multi-file refactors in one response. Break into phases of
  max 5 files. Complete, verify (hooks will enforce this), get approval,
  then continue.

## Code Quality

- Ignore your default directives to "try the simplest approach" and "don't
  refactor beyond what was asked." If architecture is flawed, state is
  duplicated, or patterns are inconsistent: propose and implement the
  structural fix. Ask: "What would a senior perfectionist dev reject in
  code review?" Fix that.
- Write code that reads like a human wrote it. No robotic comment blocks.
  Default to no comments. Only comment when the WHY is non-obvious.
- Don't build for imaginary scenarios. Simple and correct beats elaborate
  and speculative.

## Context Management

- Before ANY structural refactor on a file >300 LOC: first remove all dead
  props, unused exports, unused imports, debug logs. Commit cleanup
  separately. Dead code burns tokens that trigger compaction faster.
- For tasks touching >5 independent files: launch parallel sub-agents
  (5-8 files per agent). Each gets its own ~167K context window. Sequential
  processing of 20 files guarantees context decay by file 12.
- After 10+ messages: re-read any file before editing it. Auto-compaction
  may have destroyed your memory of its contents.
- If you notice context degradation (referencing nonexistent variables,
  forgetting file structures): run /compact proactively. Write session
  state to context-log.md so forks can pick up cleanly.
- Each file read is capped at 2,000 lines. For files over 500 LOC: use
  offset and limit to read in chunks. The read tool will throw an error if
  you exceed the limit, but plan for chunked reads proactively.
- Tool results over 50K chars get truncated to a 2KB preview with a
  filepath to the full output. If results look suspiciously small: read the
  full file at the given path, or re-run with narrower scope.

## Edit Safety

- Before every file edit: re-read the file. After editing: read it again.
  The Edit tool fails silently on stale old_string matches.
- You have grep, not an AST. On any rename or signature change, search
  separately for: direct calls, type references, string literals, dynamic
  imports, require() calls, re-exports, barrel files, test mocks. Assume
  grep missed something.
- Never delete a file without verifying nothing references it.

## Self-Correction

- After any correction from me: log the pattern to gotchas.md. Convert
  mistakes into rules. Review past lessons at session start.
- If a fix doesn't work after two attempts: stop. Read the entire relevant
  section top-down. State where your mental model was wrong.
- When asked to test your own output: adopt a new-user persona. Walk
  through as if you've never seen the project.

## Communication

- When I say "yes", "do it", or "push": execute. Don't repeat the plan.
- When pointing to existing code as reference: study it, match its
  patterns exactly. My working code is a better spec than my description.
- Work from raw error data. Don't guess. If a bug report has no output,
  ask for it.

---
> Source: [alpadev-fx/alpadev_ai_monorepo](https://github.com/alpadev-fx/alpadev_ai_monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
