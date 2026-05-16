## opencraft-crm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This is a **pre-sale planning repository** for **Ortho CRM** — an orthodontic-specific CRM platform. All 21 services (12 platform + 8 CRM + 1 frontend) are implemented. New services are built using the Ralph autonomous agent loop (see below).

## Key Documents

- `docs/00-prd-1.md` — Full PRD (v1.0, March 2026). Authoritative source for product decisions.
- `docs/01-platform-arch-design.md` — Platform architecture design (Draft, March 2026). Authoritative source for technical decisions.
- `docs/superpowers/specs/` — Component-level design specs (naming: `YYYY-MM-DD-{component}-design.md`). Authoritative for their named component once status is **Approved**.
  - `2026-03-24-automation-engine-design.md` — **Approved**
  - `2026-03-25-nurturing-engine-design.md` — **Approved**
  - `2026-03-25-messaging-service-design.md` — **Approved**
  - `2026-03-25-email-service-design.md` — **Approved**
  - `2026-03-25-notification-service-design.md` — **Approved**
  - `2026-03-25-template-service-design.md` — **Approved**
  - `2026-03-25-audience-engine-design.md` — **Approved**
  - `2026-03-25-ai-service-design.md` — **Approved**
  - `2026-03-25-analytics-service-design.md` — **Approved**
  - `2026-03-25-integration-hub-design.md` — **Approved**
  - `2026-03-25-media-service-design.md` — **Approved**
  - `2026-03-25-identity-service-design.md` — **Approved**
  - `2026-03-25-lead-service-design.md` — **Approved**
  - `2026-03-25-pipeline-engine-design.md` — **Approved**
  - `2026-03-25-conversation-service-design.md` — **Approved**
  - `2026-03-25-campaign-service-design.md` — **Approved**
  - `2026-03-25-reporting-service-design.md` — **Approved**
  - `2026-03-25-referral-service-design.md` — **Approved**
  - `2026-03-25-data-import-service-design.md` — **Approved**
  - `2026-03-25-crm-api-gateway-design.md` — **Approved**
  - `2026-03-25-crm-web-app-design.md` — **Approved**
- `docs/memories/` — Per-component key API/design decisions distilled from spec sessions (one file per service). Read the relevant file before working on or discussing a component.

## Architecture

Two-layer SOA with 21 independently deployable services in a Turborepo monorepo:

**Platform Layer (12 services)** — domain-agnostic, reusable across future products:
Messaging (Twilio), Email (SendGrid), Notification (SSE + Redis pub/sub), Template, Nurturing Engine (drip sequences), Automation Engine (event-driven workflows), Audience Engine (segment evaluation), AI (Claude API gateway), Analytics, Integration Hub (Google Ads + Meta), Identity (auth/RBAC), Media (S3/CloudFront)

**Product Layer — Ortho CRM (8 services)** — consume platform via REST + events:
Lead Service (core entity), Pipeline Engine (state machine), Conversation Service (SMS inbox), Campaign Service (email broadcasts), Referral Service, Reporting Service, Data Import Service (Ortho2 CSV), CRM API Gateway

**Frontend (1 app):** React 18 + TypeScript SPA at `apps/crm/web`. Embeds platform UI components (`@platform/template-ui`, `@platform/sequence-ui`, `@platform/audience-ui`, `@platform/automation-ui`) as React packages.

### Monorepo Structure

```
ortho/
├── apps/
│   ├── platform/        # 12 platform services (messaging, email, notification, etc.)
│   └── crm/
│       ├── lead/
│       ├── pipeline/
│       ├── conversation/
│       ├── campaign/
│       ├── referral/
│       ├── reporting/
│       ├── import/
│       ├── api-gateway/
│       └── web/         # React SPA
├── packages/
│   ├── @ortho/types         # shared TS interfaces for events + API contracts
│   ├── @ortho/event-bus     # typed EventBridge client (MockDriver available for tests)
│   ├── @ortho/auth-middleware
│   ├── @ortho/interpolator  # template variable interpolation
│   ├── @ortho/logger        # Pino, Datadog-compatible
│   ├── @platform/filter-engine  # shared pure-function filter evaluator (Automation + Audience engines)
│   └── @platform/audience-ui    # Audience segment builder React component
└── infra/               # IaC (AWS CDK or Terraform)
```

Each service follows: `src/{routes,services,repositories,events,queue}/` + `migrations/` + `test/{unit,integration}/` + `Dockerfile`

### Communication Patterns

**Async — AWS EventBridge** for state-change propagation (no direct coupling between publisher and consumer).

**Sync — REST** for queries and immediate commands (e.g. `POST /templates/render`, `POST /messages/send`, `POST /ai/complete`).

### Golden Rules (from arch doc)

1. Each service owns its DB schema — no cross-service table reads, all access through APIs or events.
2. Platform services never import product types — Automation Engine receives generic `{ entity_type, entity_id, event_type, payload }`.
3. Pipeline Engine only manages state — emits events; Automation Engine acts.
4. Platform UIs (`@platform/*`) call their own service's API directly from the browser (not proxied through CRM API Gateway). Auth uses the same Identity Service JWT.

## Local Development

Infrastructure (Postgres, Redis, Supabase Auth, MailHog) runs via Docker Compose. Before first run, generate crypto keys:

```bash
./scripts/dev/gen-keys.sh   # writes .env from .env.example, generates JWT secrets
./scripts/dev/up.sh          # start infrastructure only
./scripts/dev/up-all.sh      # start infrastructure + all services
./scripts/dev/down.sh        # stop everything
./scripts/dev/reset.sh       # wipe volumes and restart
./scripts/dev/logs.sh        # tail service logs
```

Each service reads from the root `.env` file. Copy `.env.example` → `.env` and fill in API credentials (Twilio, SendGrid, etc.) before running.

## Development Commands

Services are standalone — there is no root package.json or monorepo script runner. Run commands from inside the service directory (e.g. `apps/platform/automation/`):

```bash
npm run build        # tsc compile to dist/
npm run dev          # tsx watch (hot reload)
npm run typecheck    # tsc --noEmit
npm run test         # vitest run (all tests, single pass)
npm run test:watch   # vitest (interactive watch mode)
```

**Run a single test file:**
```bash
npx vitest run test/unit/condition-evaluator.test.ts
```

Tests live in `test/unit/` and `test/integration/` within each service.

## Ralph Autonomous Agent Workflow

Services are built via Ralph — an autonomous AI coding loop. The workflow for a new service:

1. **Generate clarifying questions** (saved to `tasks/prd-questions-<feature>.md`), fill in answers.
2. **Generate PRD** from answered questions (saved to `tasks/prd-<feature>.md`).
3. **Convert PRD to `prd.json`** using the `ralph` skill — this is the task list Ralph executes.
4. **Run Ralph:** `./scripts/ralph/ralph-cc.sh [max_iterations]`

Ralph picks the highest-priority story where `passes: false`, implements it, runs `typecheck` + `test`, commits, marks the story done in `prd.json`, and repeats. Learnings are appended to `scripts/ralph/progress.txt`.

The `run.sh` in the project root shows the pattern used for phased implementations (multiple `claude -p` invocations per phase).

## QA Testing

End-to-end QA scenarios (REST API level, ~170+ cases across 16 functional categories) are documented in:

- `docs/development/testing/qa-scenarios.md` — authoritative scenario catalogue

Execution tooling lives under `tools/qa/` — see README for the scenario runner + `/qa` Claude skill usage.

**Test priority order** (highest first): RBAC/Access Control → Lead Creation & Attribution → New Patient Pipeline → Ortho2 CSV Import → Nurture Sequences → Shared Inbox → External Integration Failures → Analytics & Attribution → Deduplication → In Treatment Pipeline → Audit & Concurrency → In Retention → AI Communications → Email Campaigns → Referral Tracking → Multi-location

## Developer Tools

Standalone utilities live under `tools/` (not part of the service monorepo — each has its own `package.json`):

| Tool | Purpose |
|------|---------|
| [tools/crm-cli](tools/crm-cli/README.md) | `crm` CLI for debugging leads, pipeline, conversations via the API Gateway |
| [tools/qa](tools/qa/README.md) | Scenario-based QA runner + health checker; paired with the `/qa` Claude skill for a fix-retest loop against the running stack |

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + TypeScript, Tailwind CSS, React Query |
| Backend | Node.js 24, TypeScript 5 (ESM — `"type": "module"`), Fastify 5 |
| ORM / Migrations | Knex 3 + pg |
| Schema validation | @sinclair/typebox 0.34 |
| Testing | Vitest 2 |
| Database | PostgreSQL (AWS RDS Multi-AZ) — shared cluster, one schema per service |
| Auth | Supabase Auth / Auth0, RBAC, SSO with EHR |
| SMS/Voice | Twilio |
| Email | SendGrid |
| AI | Claude Sonnet 4.6 (complex tasks) / Haiku 4.5 (high-volume) |
| Ads APIs | Google Ads API, Meta Marketing API |
| Event bus | AWS EventBridge |
| Job queue | BullMQ (Redis) — used by Automation Engine and Nurturing Engine for action dispatch and delayed step scheduling; Notification Service for TTL cleanup; Audience Engine for snapshot cleanup; Analytics Service for monthly partition maintenance; Identity Service for daily refresh token cleanup; Lead Service for SQS event worker |
| Infra | AWS us-east-1 (ECS Fargate, RDS, S3, CloudFront) |
| Monitoring | Datadog (APM, structured logs) |
| Monorepo | Turborepo |
| CI/CD | GitHub Actions |

## Core Product Concepts

**Three Patient Pipelines:**
1. New Patient (7 stages): New Lead → Contacted → Exam Scheduled → Exam Completed → Tx Presented → Contract Signed → Lost
2. In Treatment (3 stages): New Patient → In Treatment → Treatment Complete
3. In Retention (3 stages): Active Retention → Recall Due → Long-term Follow

**Roles:** Call Center Agent, Call Center Manager, Marketing Staff, Marketing Manager

**Lead channels:** Website forms, Google Ads, Facebook/Instagram Lead Ads, Twilio call tracking, referral links, walk-in/manual, chat widgets, Google Business Profile, CSV bulk import

**Key constraints:**
- No PHI at launch — leads are prospective patients, non-HIPAA initially
- Multi-location native (34 locations)
- Primary KPI: Cost per case start
- EHR integration is future (Ortho2 CSV bridge is temporary)

## Coding Principles

Behavioral guidelines to reduce common LLM coding mistakes.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---
> Source: [twelvedevs/opencraft-crm](https://github.com/twelvedevs/opencraft-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
