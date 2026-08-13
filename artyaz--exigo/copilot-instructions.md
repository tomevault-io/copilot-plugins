## exigo

> This file provides guidance to WARP (warp.dev) when working with code in this repository.

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Build & Dev Commands

```bash
npm run dev              # Next.js dev server with Turbopack (http://localhost:3000)
npx convex dev           # Convex backend dev server (run in separate terminal)
npm run build            # Production build
npm run check            # Lint (next lint) + type check (tsc --noEmit) — run before pushing
npm run test             # Run all tests (vitest run)
npm run test -- path/to/file.test.ts  # Run a single test file
npm run test:watch       # Watch mode
npm run test:coverage    # Tests with V8 coverage
npm run lint:fix         # Auto-fix lint issues
npm run format:write     # Format with Prettier
npm run db:studio        # Open Prisma Studio
```

CI runs `npm run check` then `npm run test:coverage`. Always run `npm run check` locally before pushing.

## Architecture Overview

This is a **T3 Stack** (Next.js 15 App Router + tRPC + Prisma) extended with **Convex** as the primary real-time database and **Google Gemini** for AI features.

### Dual Database System

- **Convex** (`convex/`) — Primary database for all application data. Schema defined in `convex/schema.ts`. All domain entities (spaces, tests, questions, knowledge nodes, courses, subscriptions, usage) live here. Convex functions (queries, mutations, actions) serve as the backend API for most operations.
- **Prisma/SQLite** (`prisma/`) — Minimal; currently only holds a legacy `Post` model from T3 scaffolding. Not used for core features.

### Frontend → Backend Data Flow

Two distinct paths exist for frontend-to-backend communication:

1. **Convex React client** — Used for real-time reactive queries/mutations from React components. Authenticated via `ConvexProviderWithClerk` in `src/app/ConvexClientProvider.tsx`. Most CRUD operations go through this path.
2. **Next.js API route handlers** (`src/app/api/`) — Used for AI-heavy operations that need SSE streaming (test generation, answer validation, lesson teaching, AI chat). These routes authenticate via Clerk's `auth()`, then create an authenticated `ConvexHttpClient` via `src/lib/convexClientAuth.ts` to call Convex functions server-side.

### AI Integration Pattern

Two call sites (intentionally separate runtimes — do not force-share clients across the Convex/Node boundary):

| Path | Where | How |
|------|--------|-----|
| **Next API routes** (`src/app/api/**`) | Streaming SSE (teach, clarify, playground generate, …) | Prefer `resolveAiProvider` / `defaultGeminiProvider` from `src/server/ai/` — honors user custom OpenAI-compatible settings when present; otherwise Gemini. Tutor tool-calling still uses env Gemini for function-calling + embeddings. |
| **Convex actions** (`convex/courseAi.ts`, `knowledgeNodesActions`, …) | Backend-only generation (module syllabus, improvements, …) | Direct `@google/genai` + `GOOGLE_GEMINI_API_KEY` / `GEMINI_MODEL` — no user BYOK; Convex cannot import `src/server` |

Shared conventions on both paths:
- Prompts are stored in the Convex `prompts` table and fetched via `convex/coursePrompts.ts` (`getPrompt`/`renderPrompt` with `{{variable}}` placeholders)
- Client-facing streams use SSE (`text/event-stream`); majority dialect is `data: {"type":"delta"|"done"|"error",...}` (see `src/lib/sse.ts` + client `src/lib/sseClient.ts`). Tutor residual uses named `event:` lines.
- AI observability events are captured via `shared/posthogAiObservability.ts` (`captureAiGenerationEvent`) where the call site supports it
- Model fallback: primary model from `GEMINI_MODEL` env var, defaults to `gemini-3-flash-preview`
- Rate limit retries: 429 responses trigger up to 3 retries with exponential backoff on the Next path

Do **not** grow a third AI entry style. Prefer extending `src/server/ai` for Next routes and the existing Gemini helpers in Convex actions.

### Auth & Authorization

- **Clerk** handles authentication. Middleware in `src/middleware.ts` protects `/spaces` and `/tests` routes.
- **Convex auth decorators** (`convex/authDecorators.ts`) provide `getAuthedContext()` / `withAuth()` which resolve the user's identity, access level, and plan limits into an `AuthedContext` object. Use these in every Convex function that needs auth.
- Three access levels: `FREE (0)`, `PRO_SCHOLAR (1)`, `EDUCATOR (2)` — defined in `convex/subscriptionService.ts`.
- Feature gating uses `requireProAccess()` / `requireEducatorAccess()` from auth decorators.

### Subscription & Plan Limits

- Plan config lives in `shared/planConfig.ts` (shared between frontend and Convex backend). **`LIMITS_BY_TIER` is the SSOT** for numeric entitlements (spaces, tests/month, knowledge pieces, deep dives). `getMarketingPerksForTier` derives seed + pricing perk strings from the same table so marketing cannot drift from enforcement.
- `convex/subscriptionService.ts` maps access levels → tiers via thin `getLimitsForAccessLevel` → `getLimitsForTier` (no Strategy classes). Access levels remain `FREE (0)` / `PRO_SCHOLAR (1)` / `EDUCATOR (2)`.
- Usage tracking (tests, deep dives) is **calendar-month entity counts** in `convex/tests.ts` / `convex/deepDives.ts`. There is no separate rolling-window usage meter (`usageService` was removed).
- Plan catalog seed is `convex/seedPlans.ts` (internal). `seed` refuses re-seed if rows exist. To fix stale free “10 AI tests” copy without deleting rows, run internal `seedPlans.syncPerksFromSsot` (patches every plan’s `perks` from `getMarketingPerksForTier`).
- Payments are handled via **Paddle** through the provider abstraction in `src/server/payments/` (`IPaymentProvider` interface, `PaddleProvider` implementation).

### Adaptive Course System (Educator-only)

The course system in `convex/` uses a state machine orchestrated by `convex/courseOrchestrator.ts`:
- Phases: `baseline` → `module_generation` → `lesson` → `lesson_summary` → `module_complete` → (loop or `completed`)
- `courseAi.ts` generates modules/lessons via Gemini
- `courseLessons.ts` tracks lesson status: `pending` → `goals_set` → `teaching` → `completed` → `summarized` → `integrated`
- Lesson teaching is streamed via `src/app/api/learn/teach/route.ts`

### Shared Code

The `shared/` directory contains code imported by both the Next.js frontend and the Convex backend. This is necessary because Convex functions run in a separate runtime and cannot import from `src/`. When adding constants, types, or utilities needed on both sides, put them here.

## Path Aliases

- `~/` maps to `./src/` (configured in `tsconfig.json`)
- Convex generated types: `convex/_generated/` (auto-generated, do not edit)

## Testing

Tests use **Vitest** with `globals: true` (no need to import `describe`/`it`/`expect`). Test files follow `*.test.ts` / `*.spec.ts` patterns. Tests cover Convex backend logic and shared utilities — they run in Node environment, not browser.

## Git Commit Style

Commit messages should sound human and informal — lowercase, conversational, occasionally clumsy. Avoid strict conventional commit formatting. Examples: "fix typo in the header", "finally fixed the stupid bug in auth", "various little fixes".

## Environment Setup

Required env vars: `NEXT_PUBLIC_CONVEX_URL`, `CONVEX_DEPLOYMENT`, `GOOGLE_GEMINI_API_KEY`. Clerk runs in keyless dev mode locally. Copy `.env.example` to `.env` and fill in values. Use `SKIP_ENV_VALIDATION=1` to bypass env validation during builds (used in CI).

---
> Source: [artyaz/exigo](https://github.com/artyaz/exigo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
