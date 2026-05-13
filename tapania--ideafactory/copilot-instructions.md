## ideafactory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development
pnpm dev                          # Start all packages in parallel (server:3000, client:5173)
pnpm run --filter server cli      # Run CLI: pnpm run --filter server cli -- run "domain"

# Build & Typecheck
pnpm build                        # Build all packages
pnpm typecheck                    # Typecheck all packages

# Testing (vitest workspace with projects: shared, server, client)
pnpm test                         # Run all tests once
pnpm test:server                  # Run server tests only
pnpm test:client                  # Run client tests only
pnpm test:shared                  # Run shared tests only
pnpm vitest run path/to/file      # Run a single test file

# Linting
pnpm lint                         # ESLint (flat config, ESLint 9)
```

## Architecture

IdeaFactory is an AI-powered ideation pipeline that uses Claude to generate, evaluate, evolve, and package creative ideas. It is a **pnpm monorepo** with three packages.

### Package Dependency Graph

```
@ideafactory/shared          ← Zod schemas, constants, types (leaf package)
    ↑             ↑
@ideafactory/server          @ideafactory/client
(Hono + tRPC + SQLite +      (React 19 + Vite + Tailwind +
 Anthropic SDK + CLI)          Zustand + tRPC client)
```

The client imports only the `AppRouter` **type** from server (devDependency) for tRPC type safety. No runtime server code is imported by the client.

### Communication

- **tRPC** (`/trpc/*`): Client↔Server RPC for mutations and queries. Uses `superjson` transformer and `httpBatchLink`. Vite proxies `/trpc` and `/api` to `localhost:3000` in dev.
- **SSE** (`/api/session/:id/stream`): Server→Client real-time events during pipeline execution. The Zustand store's `handleSSEEvent()` dispatches all 11 SSE event types.

### Pipeline Stages

The system follows a staged pipeline orchestrated in `packages/server/src/agents/pipeline.ts`:

```
taxonomy → methods → rubric → factory → completed
```

| Stage | Agent | Model (default) | Temp | Key File |
|-------|-------|-----------------|------|----------|
| Taxonomy | Navigator | claude-opus-4-6 | 0.7 | `agents/navigator.ts` |
| Methods | Strategist | claude-opus-4-6 | 0.6 | `agents/strategist.ts` |
| Rubric | Strategist | claude-opus-4-6 | 0.6 | `agents/strategist.ts` |
| Factory | Workers + Analyst | claude-opus-4-6 | 0.5–0.9 | `agents/factory.ts` |

The Factory stage has automated sub-phases: **Diverge** (one worker per selected method, on-the-fly personas from method fields, temp=0.9) → **Converge** (batched scoring in groups of 5, gate filtering + top-N selection, temp=0.5) → **Evolve** (pair-based cross-pollination with re-scoring, returns survivors + evolved combined, temp=0.7) → **Interactive** (user selects ideas for QA and packaging via tRPC mutations).

Interactive operations (triggered by user, not automated):
- **Critical Review** (`session.runCriticalReview`): chains QA (parallel per-idea feasibility/risk agents → `qa_sheets`) then packaging (per-idea HTML artifact + deep research prompt → `idea_packages`) server-side
- **Complete** (`session.completeSession`): marks session as completed

### Agent System

- Each agent uses a **skill markdown file** from `packages/server/src/skills/` as its system prompt, loaded via `fs.readFileSync` at module init.
- LLM calls go through `packages/server/src/agents/llm.ts` which provides `callLLM()`, `callLLMWithRetry()` (with Zod validation, exponential backoff for 429s, JSON repair on retries), and `extractJSON()`.
- Models are configurable per role via `~/.ideafactory/config.yaml`.

### Data Layer

- **SQLite** via better-sqlite3 + drizzle-orm, stored at `~/.ideafactory/data.db`
- 8 tables: `sessions`, `taxonomy_trees`, `method_selections`, `rubrics`, `ideas`, `qa_sheets`, `idea_packages`, `event_log`
- Schema defined in `packages/server/src/db/schema.ts`, inline migrations (CREATE TABLE IF NOT EXISTS)

### Client Architecture

- **Zustand** store (`packages/client/src/store/index.ts`) holds all session state and dispatches SSE events
- Stage-specific UI components in `packages/client/src/components/stages/`
- Dark theme with accent `#6e56cf`, fonts: Inter + JetBrains Mono

### Configuration

Configuration is resolved in priority order: **session config > .env / env vars > config.yaml > defaults**.

- **`.env`** at project root (loaded via `dotenv`): see `.env.example` for all supported vars
  - `ANTHROPIC_API_KEY` — or use Claude Code session auth
  - `MODEL_DEFAULT`, `MODEL_NAVIGATOR`, `MODEL_STRATEGIST`, `MODEL_WORKER`, `MODEL_ANALYST` — per-role model assignment
  - `HAIKU_MODEL`, `SONNET_MODEL`, `OPUS_MODEL` — model version IDs for each tier (served to client for model selector)
  - `PORT`, `IDEAS_PER_WORKER` (5-30), `WEB_SEARCH` (true/false)
- **`~/.ideafactory/config.yaml`** — optional per-user config (same keys as env vars, YAML format)
- **Defaults** in `packages/shared/src/constants.ts` (`DEFAULT_CONFIG`)
- Custom methods: YAML/JSON in `~/.ideafactory/methods/`
- Worker count is determined by the number of selected methods (one worker per method)

## Code Style

- Prettier: single quotes, trailing commas, semicolons, 100 char width, 2-space indent
- ESLint: `@typescript-eslint/no-unused-vars` (warn, `_` prefix ignored), `@typescript-eslint/no-explicit-any` (warn)
- TypeScript strict mode, ES2022 target, bundler module resolution

## Testing

- Vitest workspace with three projects (server: node env, client: jsdom env, shared: node env)
- Server test setup in `packages/server/src/__tests__/setup.ts` provides `createTestDb()` (in-memory SQLite) and `mockAnthropicSdk()`
- All test configs enable `globals: true` (no need to import `describe`/`it`/`expect`)

---
> Source: [tapania/ideafactory](https://github.com/tapania/ideafactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
