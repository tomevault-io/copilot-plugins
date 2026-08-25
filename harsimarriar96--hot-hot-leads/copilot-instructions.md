## hot-hot-leads

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**HotHotLeads** is an AI-led GTM (Go-To-Market) Engineering engine built on NestJS + Claude. It:
- Reads and writes product knowledge (ICP, strategy, channels, pricing) in a **Postgres** database managed via MikroORM in [packages/db](packages/db/)
- Discovers, enriches, scores leads and drafts outreach using **Claude multi-agent pipelines**
- Monitors GTM signals via **Exa** web research
- Is controlled via **Telegram** and **WhatsApp** — both on-demand and on schedule
- Persists all output (leads, outreach, signals, feedback) in Postgres

## Commands

All commands run from the repo root and are orchestrated by Turborepo.

```bash
# Install all workspace dependencies (one node_modules at the root)
npm install

# One-time: start Postgres and apply migrations
docker compose up -d postgres
npm --workspace @hothotleads/db run db:migrate

# Development — starts both apps in watch mode
npx turbo run dev

# Run only the API in watch mode
npx turbo run dev --filter=@hothotleads/api

# Run only the web app
npx turbo run dev --filter=@hothotleads/web

# Production build (packages, then apps, in dependency order)
npx turbo run build

# Type-check the entire workspace
npx turbo run typecheck

# Lint the entire workspace
npx turbo run lint

# Run all tests
npx turbo run test

# Run a single API test file
npm --workspace @hothotleads/api run test:single -- src/agents/orchestrator/orchestrator.agent.spec.ts

# Start the API from its build output
npm --workspace @hothotleads/api run start
```

## Architecture

This is a Turborepo monorepo with apps under `apps/` and shared packages under `packages/`.

```
apps/
├── api/                            # NestJS API + agent orchestration (@hothotleads/api)
│   ├── package.json
│   ├── tsconfig.json               # extends @hothotleads/config/tsconfig/nest.json
│   ├── nest-cli.json
│   ├── .env                        # runtime secrets (gitignored)
│   └── src/
│       ├── main.ts                 # NestJS bootstrap
│       ├── app.module.ts           # Root module wiring
│       ├── health.controller.ts    # GET /health
│       ├── config/configuration.ts # All env vars typed
│       ├── common/types/           # Shared TypeScript interfaces
│       ├── agents/                 # BaseAgent + 8 specialized agents
│       ├── channels/               # Telegram + WhatsApp webhook controllers
│       ├── integrations/           # Exa / Tavily / Perplexity web-research clients
│       ├── db/db.module.ts         # MikroORM wiring + repository providers
│       └── scheduler/              # Cron-registration service (loads schedules from Postgres)
└── web/                            # Vite + React operator UI (@hothotleads/web)
    ├── package.json
    ├── tsconfig.json               # extends @hothotleads/config/tsconfig/vite.json
    ├── vite.config.ts
    └── src/                        # pages, components, hooks

packages/
├── config/                         # Shared TSConfig / ESLint / Prettier bases (@hothotleads/config)
│   ├── tsconfig/{base,nest,vite,library}.json
│   ├── eslint/base.js
│   └── prettier/index.js
├── types/                          # Shared domain types (@hothotleads/types) — plain interfaces shared between api + web
└── db/                             # Postgres schema + entities + repositories (@hothotleads/db)

turbo.json                          # pipeline: build, dev, lint, typecheck, test, start
```

**Dependency rule:** apps may depend on packages. Packages must not depend on apps. Apps must not import from sibling apps.

## Agent Architecture

8 specialized agents, all extending `BaseAgent` (which owns the Claude tool-use loop):

| Agent | Role | Key Tools |
|-------|------|-----------|
| **OrchestratorAgent** | Entry point — parses intent, loads product context, routes to specialists | get_products, load_product_context, run_* |
| **ProductContextAgent** | Loads + summarizes a product's knowledge (ICP, channels, sources, predefined prospects) from Postgres | get_product_knowledge |
| **ProspectDiscoveryAgent** | Finds new companies matching ICP via Exa | search_companies, save_lead |
| **EnrichmentAgent** | Deep-researches a company (news, funding, hiring, tech stack) | research_company, search_web, update_lead |
| **ScoringAgent** | Scores leads 0–100 against ICP + signals | get_product_knowledge, update_lead_score |
| **OutreachAgent** | Drafts channel-specific personalized messages referencing signals | get_lead, save_outreach |
| **SignalMonitorAgent** | Monitors Sources for new GTM signals and writes them to Notion | search_signals, save_signal |
| **FeedbackAgent** | Evaluates output quality (0–10) and writes to Feedback Log | get_recent_feedback, save_feedback |

### Feedback Loop
Every significant output (lead batch, outreach draft) automatically triggers **FeedbackAgent** via `OrchestratorAgent.executeTool('run_feedback', ...)`. Feedback is persisted in the `feedback_entries` table (scoped by `product_id`). Run `/feedback-review` to surface patterns and ICP refinement suggestions.

## Data Model

Postgres is the single source of truth. Schema lives in [packages/db/src/entities/](packages/db/src/entities/).

```
Product (root of the GTM domain)
├── ProductIcp                 1:1  — target industries, sizes, funding stages, geographies, notes
├── ProductSalesChannel        1:N  — named outreach channels with templates
├── ProductSource              1:N  — signal sources (news, social, databases…)
├── PredefinedProspect         1:N  — seed companies for prospect discovery
├── Lead                       1:N
│     └── OutreachDraft        1:N  — drafts per lead, multiple channels
├── Signal                     1:N  — buying intent / hiring / activity signals
├── Schedule                   1:N  — recurring tasks delivered via Telegram/WhatsApp
└── FeedbackEntry              1:N  — agent quality scores
```

Product-owned tables cascade on delete. `leads(product_id, domain)` has a partial unique index (only when `domain IS NOT NULL`) to prevent dupes. UUID primary keys come from `pgcrypto.gen_random_uuid()`.

Repositories in [packages/db/src/repositories/](packages/db/src/repositories/) wrap the `EntityManager` — `ProductsRepository`, `LeadsRepository`, `SignalsRepository`, `SchedulesRepository`, `FeedbackRepository`. Apps consume them as Nest providers via [apps/api/src/db/db.module.ts](apps/api/src/db/db.module.ts).

## Local dev — database

```bash
docker compose up -d postgres                           # Postgres 16 on localhost:5434
npm --workspace @hothotleads/db run db:migrate          # apply migrations
```

The API fails fast at boot if `DATABASE_URL` is missing or unreachable.

## Product chats

Each product has a single, persistent chat thread rooted on `product_chats` + `product_chat_messages`. Messages are append-only and store Anthropic-shaped `content_blocks` (text / tool_use / tool_result) as JSONB.

- **Web chat surface:** `/products/:productId` Chat tab. `POST /products/:productId/chat/messages` returns Server-Sent Events (`message_start`, `text_delta`, `tool_use_start`, `tool_use_result`, `message_done`, `error`). The client renders text incrementally and discloses tool calls inline.
- **Streaming agent:** [BaseAgent.runStream](apps/api/src/agents/base/base.agent.ts) maps the Claude Agent SDK's async iterator into the event types above. Non-streaming `run()` is a thin wrapper that drains the generator.
- **Channel sync:** `ChatService.handleIncoming` is used by both Telegram and WhatsApp handlers. When `workspaces.sync_channel_chats = true` (default) and a product is resolved, the channel exchange is persisted to the same product thread with `source_channel` set to `telegram` or `whatsapp`.
- **Product management:** `/api/products` — list / get / create / patch / delete. Delete cascades to leads, outreach drafts, signals, schedules, feedback, and chat messages via the DB FKs.
- **Client disconnect:** the SSE handler continues the orchestrator run to completion even if the client aborts mid-stream, so the assistant message is always persisted in full. A subsequent `GET /products/:productId/chat` shows the complete reply.

### Known scope calls

- The Overview tab edits basics (name / generalInfo / salesStrategy / pricing). ICP, sales-channel, source, and predefined-prospect editing is **display-only in v1**; editing those nested collections comes in a follow-up.
- Telegram/WhatsApp messages that don't resolve to a product fall back to running the orchestrator directly without persistence. A "product selection" convention for channels is a future concern.

## Onboarding

A fresh deploy has an empty database. The operator bootstraps the system via an in-app wizard at `/onboarding` (4 steps: workspace details → operator profile → channels → first product). Progress is persisted as `workspaces.onboarding_status` so reloads resume where you left off.

- **Entities:** `Workspace` (singleton, enforced by a unique partial index on `singleton = TRUE`), `Operator`, `ChannelCredential`.
- **Channel credentials** are stored encrypted at rest (AES-256-GCM, key = SHA-256 of `APP_SECRET_KEY`). The plaintext never leaves the server process; the API exposes only redacted `{ channel, configured, updatedAt }` views.
- **Telegram** credentials are validated via a live `getMe` call before being saved. **WhatsApp** credentials pass a shallow shape check (full validation happens on first outbound message).
- **State machine:** `pending → workspace_details → operator_profile → channels → first_product → completed`. Out-of-order submissions return `409 Conflict`.
- **Admin reset:** `POST /onboarding/reset` with header `X-Admin-Token: $APP_SECRET_KEY` wipes the workspace + operator + channel rows. Not exposed in the web UI by design.
- **Settings surface:** after onboarding, edits go through `GET/PATCH /settings` and the corresponding web page at `/settings`.

## Environment Variables

```
ANTHROPIC_API_KEY, CLAUDE_MODEL, CLAUDE_MODEL_HEAVY, CLAUDE_MODEL_FAST
TELEGRAM_BOT_TOKEN, TELEGRAM_WEBHOOK_SECRET
WHATSAPP_PHONE_NUMBER_ID, WHATSAPP_ACCESS_TOKEN, WHATSAPP_VERIFY_TOKEN
DATABASE_URL                  # postgresql://user:pass@host:port/db — required
DATABASE_SSL                  # true|false, default false
MIKROORM_DEBUG                # true|false, default false
APP_SECRET_KEY                # required, min 32 chars. Encrypts channel credentials + guards the onboarding reset endpoint. Generate: `openssl rand -hex 32`
EXA_API_KEY, TAVILY_API_KEY, PERPLEXITY_API_KEY
HOST, PORT, PUBLIC_URL, LOG_LEVEL
```

## Slash Commands (`.claude/commands/`)

| Command | What it does |
|---------|-------------|
| `/generate-leads` | Full pipeline: discovery → enrichment → scoring → feedback for a product |
| `/check-signals` | Monitor Sources via Exa, classify + save new signals |
| `/draft-outreach` | Write channel-specific personalized outreach for a lead |
| `/feedback-review` | Aggregate Feedback Log — quality trends + ICP improvement suggestions |
| `/setup-schedule` | Create a recurring task delivered to Telegram/WhatsApp |

## Key Design Decisions

- **`BaseAgent` is the only place the Anthropic SDK is called** — every specialized agent inherits the tool-use loop. To add a new agent: extend `BaseAgent`, define `systemPrompt`, `tools[]`, and `executeTool()`.
- **Product knowledge is loaded via a single query.** `ProductsRepository.getProduct(name)` returns the Product with its ICP, sales channels, sources, and predefined prospects populated in one round-trip. There is no in-process cache (Postgres reads are fast).
- **Schedules survive restarts** — `SchedulerService.onApplicationBootstrap` loads `enabled=true` schedules from Postgres and registers each `CronJob`. Registering a new schedule via chat persists to the `schedules` table and live-registers the job.
- **Feedback is automatic** — `OrchestratorAgent` always calls `run_feedback` after prospect discovery and outreach. This is intentional and should not be removed.
- **WhatsApp `chatId` == phone number** — Unlike Telegram, there is no separate chat ID.

---
> Source: [harsimarriar96/hot-hot-leads](https://github.com/harsimarriar96/hot-hot-leads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
