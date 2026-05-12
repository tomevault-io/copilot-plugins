## flowforge

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlowForge is a production-ready AI workflow orchestration platform. It enables building durable, event-driven workflows with automatic retries, step memoization, and LLM integration.

## Repository Structure

```
flowforge/
├── packages/
│   ├── flowforge-sdk/       # Python SDK for defining workflows
│   ├── flowforge-cli/       # CLI for development and management
│   └── flowforge-client-ts/ # TypeScript client for Node.js/Next.js
├── server/                  # FastAPI server for orchestration
│   ├── src/flowforge_server/
│   │   ├── api/routes/      # API endpoints (events, functions, runs, users, auth)
│   │   ├── db/models/       # SQLAlchemy models (Tenant, User, Function, Run, etc.)
│   │   └── services/        # Business logic (auth, user, AI)
│   └── migrations/          # Database migrations
├── dashboard/               # Next.js admin dashboard
│   └── src/
│       ├── app/             # Pages (login, settings, runs, functions, etc.)
│       ├── components/      # React components
│       ├── stores/          # Zustand stores (auth-store)
│       └── lib/             # Utilities, API client, auth types
├── examples/                # Example workflows
└── tests/                   # Test suites (unit, integration, e2e)
```

## Development Commands

### Python (SDK, CLI, Server)

```bash
# Install dependencies (from root)
pip install -e "packages/flowforge-sdk[all]"
pip install -e packages/flowforge-cli
pip install -e server

# Run linting
ruff check .

# Run type checking
mypy packages/flowforge-sdk/src packages/flowforge-cli/src server/src

# Run tests
pytest                           # all tests
pytest tests/unit               # unit tests only
pytest -v -k "test_name"        # specific test

# Start development server
cd examples && flowforge dev .

# Send test event
flowforge send order/created -d '{"order_id": "123", "customer": "Alice", "total": 99.99}'
```

### Dashboard (Next.js)

```bash
cd dashboard
pnpm install
pnpm dev          # development server at localhost:3000
pnpm build        # production build
pnpm lint         # run eslint
```

### Infrastructure

```bash
# Start PostgreSQL and Redis
docker-compose up -d

# PostgreSQL: localhost:5432 (user: flowforge, pass: flowforge)
# Redis: localhost:6379
```

## Architecture

### SDK Core Concepts

**FlowForge Functions**: Async functions decorated with `@flowforge.function()` that define workflows. Functions receive a `Context` and use `step.*` primitives for durable execution.

**Steps**: Durable primitives that provide memoization and checkpointing:
- `step.run(id, fn, *args)` - Execute a function with memoization
- `step.sleep(id, duration)` - Pause execution for a duration
- `step.ai(id, model=, prompt=)` - Execute LLM calls with retry
- `step.wait_for_event(id, event=, match=)` - Pause until event arrives
- `step.invoke(id, function_id=, data=)` - Call another function
- `step.send_event(id, name=, data=)` - Emit an event

**Triggers**: Define how functions are invoked:
- `flowforge.trigger.event("order/created")` - Event-based trigger
- Cron and webhook triggers also supported

**Flow Control**: Concurrency limiting, rate limiting, throttling, debouncing via configuration parameters on `@flowforge.function()`.

### Server Architecture

- **API Layer** (`server/api/`): FastAPI routes for events, functions, runs, health, users, auth
- **Core Logic** (`server/core/`): Event processing, run management, flow control
- **Services** (`server/services/`): Executor (job worker), AI service (LiteLLM integration), User service (auth)
- **Queue** (`server/queue/`): Redis-backed fair queue with tenant isolation
- **Database Models** (`server/db/models/`): Tenant, Function, Event, Run, Step, User, ApiKey

### Authentication & Authorization

FlowForge has two authentication mechanisms:

**1. Dashboard Users (email/password)**
- For human users accessing the dashboard
- Three roles with different permissions:
  - **Admin**: Full access - manage users, API keys, functions, tools, everything
  - **Member**: Create/edit functions, tools, send events, view runs, manage approvals
  - **Viewer**: Read-only access to all resources
- Uses JWT tokens stored in browser

**2. API Keys (ff_xxx)**
- For SDK/application authentication (server-to-server)
- Three types: `live` (production), `test` (development), `ro` (read-only)
- Scoped permissions (events:send, runs:read, functions:manage, etc.)
- Format: `ff_{type}_{random}` (e.g., `ff_live_a1b2c3d4...`)

**Creating the First Admin:**
```bash
cd server
flowforge-server create-admin --email admin@example.com --password secret123
# Or use the standalone command:
flowforge-create-admin -e admin@example.com -p secret123 -n "Admin User"
```

### Execution Flow

1. Event arrives at `/api/v1/events`
2. Server matches event to registered functions by trigger
3. Creates `Run` record and enqueues job to `FairQueue`
4. Executor dequeues job, invokes user function via HTTP
5. Function executes until a `step.*` call raises `StepCompleted`
6. Server saves step result, re-enqueues for continuation
7. Repeats until function returns (completes) or fails

### Key Patterns

- Steps raise control flow exceptions (`StepCompleted`, `StepFailed`) to yield control back to the server
- Memoization uses step hashes - completed steps return cached results on replay
- `_StepProxy` provides global `step` instance configured per-execution context

### Agent Team Platform (Multica-inspired)

FlowForge includes an agent team management layer where AI agents are first-class team members:

**Agents** (`/api/v1/agents`): Named AI identities with status (online/idle/busy/offline), avatar, model, system prompt, and enabled skills. Agents can be assigned to tasks and linked to functions.

**Tasks** (`/api/v1/tasks`): Kanban-style task board with statuses (todo, in_progress, in_review, done, blocked, cancelled). Tasks support priorities, labels, sub-tasks, and can be assigned to humans or agents. Human-readable identifiers (FF-1, FF-2, etc.).

**Comments** (`/api/v1/comments`): Unified activity timeline on tasks and runs. Supports user and agent authors, @mentions, emoji reactions.

**Notifications** (`/api/v1/notifications`): Inbox system for approvals, task assignments, mentions, agent blockers, run failures. Mark read/archive.

**Skills** (`/api/v1/skills`): Reusable function+tool configuration templates AND imported knowledge from external marketplaces.
- **Local skills**: Capture function configs via "Save as Skill" from the Functions page
- **Marketplace**: Search skills.sh, preview SKILL.md content, import into FlowForge
- **Runtime injection**: Skills are NOT baked into system prompts. Instead, each function/agent has an `enabled_skills[]` list. The `InlineExecutor` loads only enabled skills at runtime and wraps them in `<skill-knowledge>` tags.
- **Endpoints**: `GET /skills/marketplace/search`, `GET /skills/marketplace/preview`, `POST /skills/marketplace/import`, `POST /skills/{id}/refresh`
- **Skill toggle**: `PUT /functions/{id}/skills`, `PUT /agents/{id}/skills`

**Cost Dashboard** (`/costs`): Token usage, costs by provider/model, daily trends. Surfaces existing usage data prominently.

**Live Activity Panel**: Real-time SSE-powered panel on run detail pages showing agent thinking, tool calls, and events as they happen.

**Key Models**: Agent, Task, Comment, Notification, SkillTemplate (all in `server/src/flowforge_server/db/models/`)

**Key Services**: `skill_marketplace.py` (skills.sh HTTP client + SKILL.md parser)

## Configuration

Environment variables for the server:
- `FLOWFORGE_HOST`, `FLOWFORGE_PORT` - Server binding
- `FLOWFORGE_DATABASE_URL` - PostgreSQL connection string
- `FLOWFORGE_REDIS_URL` - Redis connection string
- `FLOWFORGE_JWT_SECRET` - Secret for JWT token signing (required for dashboard auth)
- `FLOWFORGE_OPENAI_API_KEY`, `FLOWFORGE_ANTHROPIC_API_KEY` - For AI steps (via LiteLLM)

## Testing Notes

- Tests use `pytest-asyncio` with `asyncio_mode = "auto"`
- Server tests require running PostgreSQL/Redis (use docker-compose)

---
> Source: [hoootan/flowforge](https://github.com/hoootan/flowforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
