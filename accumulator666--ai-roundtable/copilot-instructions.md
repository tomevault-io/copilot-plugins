## ai-roundtable

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Mesh is a multi-AI Docker infrastructure running a Fortune 500-style AI company. Five companies (Holding, MeshTech, MeshMedia, MeshCapital, MeshVentures) each run AI roundtables with specialized participants. A CEO can pitch a project idea to any company — the system breaks it into tasks, assigns to the right specialists, executes in parallel, and synthesizes a final deliverable. A central router dispatches requests to 5 AI provider proxies, Claude Code CLI, and a remote R730 Ollama node. Redis provides cross-company messaging, PostgreSQL stores state.

## Architecture

Five-tier system: **Proxies** normalize AI APIs -> **Router** orchestrates model routing -> **Company Roundtables** run domain-scoped AI boards with project orchestration -> **Orchestrator** coordinates cross-company projects -> **Supporting services** (Redis, PostgreSQL, n8n, Caddy).

> **Migration note:** MeshCorp HQ (8160) is the next-generation replacement for the company roundtables (8130-8134) and legacy collab-chat (8139). It provides a unified dashboard with dynamic org templates instead of fixed per-company containers. Both old and new systems coexist during migration.

```
Client / Shell
    |
Caddy (reverse proxy, TLS — ports 80/443/8444)
    |
AI Router (8110) — central orchestrator
    |--- Claude Proxy (8100) --- Anthropic API
    |--- ChatGPT Proxy (8101) --- OpenAI API
    |--- Grok Proxy (8102) --- xAI API
    |--- Gemini Proxy (8103) --- Google API
    |--- DeepSeek Proxy (8105) --- DeepSeek API
    |--- Claude Code API (8104) --- Claude CLI
    '--- R730 Ollama (10.0.0.2:11434)

MeshCorp HQ (8160) — unified dashboard, dynamic org templates

Company Roundtables (8130-8134) — Fortune 500 simulation
    |--- Holding Board (8130) — CEO, orchestrates cross-company projects
    |--- MeshTech (8131) — Engineering
    |--- MeshMedia (8132) — Marketing & Content
    |--- MeshCapital (8133) — Finance (+ Options Trader integration)
    '--- MeshVentures (8134) — R&D & Sales

Shared Infrastructure:
    PostgreSQL (ai_mesh DB via ai-stack_default network)
    Redis (pub/sub, cross-company comms)
    CrewAI Agents (8120) — autonomous background jobs
    Legacy Collab Chat (8139) — being replaced by company roundtables
    n8n (5678) — workflow automation
```

### Key Directories

- **`router/`** — Central orchestrator. `routing.py` is **source of truth** for all model->proxy mappings (40+ models), R730 model set, aliases, and auto-routing patterns.
- **`proxies/{claude,chatgpt,grok,gemini,deepseek}/`** — Thin FastAPI wrappers normalizing each provider API to OpenAI-compatible `/v1/chat/completions`. Each has `main.py` + `requirements.txt`.
- **`companies/shared/`** — Shared code for all company roundtables:
  - `base_app.py` — `create_company_app()` factory with roundtable engine, project mode, WebSocket handler
  - `orchestrator.py` — Cross-company project orchestration, `SKILL_ROUTING` (35 skill->company mappings)
  - `db.py` — Async PostgreSQL (asyncpg), auto-creates all tables on first connect
  - `company_comms.py` — Redis pub/sub for cross-company messaging
- **`companies/{holding,meshtech,meshmedia,meshcapital,meshventures}/`** — Each has `main.py` (calls `create_company_app()`), `participants.json`, `company_config.json`, `index.html`. The `shared/` symlink in each points to `companies/shared/`.
- **`meshcorp/`** — MeshCorp HQ: unified dashboard replacing company roundtables. `main.py` (FastAPI), `conversation.py` (event-driven engine), `projects.py` (lifecycle manager), `routes.py` (REST + WebSocket API), `templates/` (7 org template JSONs), `frontend/` (Svelte SPA).
- **`collab-chat/`** — Legacy monolithic roundtable (single `main.py` ~2900 lines + `index.html`). Has its own `CLAUDE.md` with detailed docs. Being replaced by MeshCorp HQ.
- **`agents/`** — CrewAI background agents. Has its own `Dockerfile`. `crew.py` defines 5 agents (strategist, researcher, builder, marketer, finance).
- **`claude-code/`** — REST wrapper for Claude Code CLI. Has its own `Dockerfile`. Uses OAuth credentials from `/root/.claude/.credentials.json`.
- **`claude-mcp/`** — MCP (Model Context Protocol) server providing delegation tools to cheaper/free models.
- **`shared/`** — Top-level shared utilities: `models.py` (Pydantic models for chat protocol), `config.py` (legacy proxy URL mappings, superseded by `router/routing.py`).

## Commands

```bash
# Start everything
docker compose up -d

# Check health of all backends
curl -s http://localhost:8110/health | jq

# Check all companies
for p in 8130 8131 8132 8133 8134; do curl -s http://localhost:$p/health | jq .company_name; done

# View logs
docker logs ai-mesh-router -f --tail 50

# Restart a service (picks up code changes — volumes are mounted)
docker restart ai-mesh-router

# Restart all company roundtables after editing companies/shared/ code
docker restart ai-mesh-holding-board ai-mesh-meshtech ai-mesh-meshmedia ai-mesh-meshcapital ai-mesh-meshventures

# Only claude-code and crewai-agents need docker compose build (they have Dockerfiles)
docker compose build claude-code && docker compose up -d claude-code
docker compose build crewai-agents && docker compose up -d crewai-agents

# Run a chat completion through the router
curl -X POST http://localhost:8110/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "auto", "messages": [{"role": "user", "content": "Hello"}]}'

# Multi-model fan-out
curl -X POST http://localhost:8110/v1/chat/multi \
  -H "Content-Type: application/json" \
  -d '{"models": ["claude-haiku-4-5", "grok-3-mini", "deepseek-chat"], "messages": [{"role": "user", "content": "Hello"}]}'

# MeshCorp HQ
curl -s http://localhost:8160/health | jq
curl -s http://localhost:8160/api/templates | jq '.templates[].name'
curl -s http://localhost:8160/api/dashboard | jq

# Database (always specify -d, default DB is 'templates' not 'ai_mesh')
docker exec -it prompt-template-db psql -U admin -d ai_mesh
```

## Testing

Integration tests run against live Docker containers — all services must be running.

```bash
# All tests
pytest tests/ -v

# Single test file
pytest tests/test_router.py -v

# Single test
pytest tests/test_router.py::test_health -v

# Collab-chat tests
pytest collab-chat/tests/ -v
```

Test fixtures in `tests/conftest.py` provide httpx clients with hardcoded `BASE_URLS` per service (e.g., `http://localhost:8110` for router, `http://localhost:8130` for holding).

## Development Patterns

**Runtime model**: All services use `python:3.12-slim` with `pip install -r requirements.txt` at container start, then `uvicorn main:app --host 0.0.0.0 --port 8000`. Every container listens on internal port 8000, mapped to different host ports. Code is volume-mounted, so `docker restart <container>` picks up changes — no rebuild needed except for `claude-code` and `crewai-agents` (which have Dockerfiles).

**Tech stack**: Python 3.12, FastAPI, uvicorn, httpx, Pydantic v2 throughout. No linting or formatting configured.

**All model calls** go through the AI Router (`ROUTER_URL`), never directly to provider APIs. Inside containers, the router URL is `http://ai-mesh-router:8000`.

**Proxy pattern**: Each proxy normalizes a provider API to OpenAI-compatible format. Key translations:
- `claude/` — Merges consecutive same-role messages (Anthropic requirement), strips trailing assistant messages
- `chatgpt/` — `max_tokens` -> `max_completion_tokens` for GPT-5 family, strips `temperature`/`top_p` for reasoning models (o3, o1)
- `gemini/` — Converts to Google `contents` structure
- `grok/`, `deepseek/` — Near-passthrough

**Company shared code**: The `companies/shared/` directory is both symlinked into each company dir AND volume-mounted in docker-compose (`./companies/shared:/app/shared`). Requirements are installed from `shared/requirements.txt`.

## Model Routing

**Source of truth**: `router/routing.py`

**Anthropic models** route through `claude-code` (Max subscription), not `claude-proxy`. If you get 401 errors on Claude models, the OAuth token expired — run `claude auth login` then `docker restart ai-mesh-claude-code`.

**Auto-routing** (`model: "auto"`): coding -> GPT-5.2, creative -> GPT-5.2, research -> Grok-3, image -> Gemini 2.0 Flash, default -> Claude Sonnet 4.5. Patterns defined in `AUTO_ROUTE_PATTERNS`.

**Model aliases** (`MODEL_ALIASES`): `:latest` tags resolve to exact R730 versions (e.g., `qwen3:latest` -> `qwen3:30b`). Always update `MODEL_ALIASES` when adding new Ollama models.

**Fallback chain**: `MODEL_TO_PROXY` -> `R730_MODELS` -> R730 Ollama. Unknown models fall through to R730 since local Ollama is not running.

**Delegation**: Models can delegate via `[DELEGATE:model_name]prompt[/DELEGATE]` tags parsed by `router/delegation.py`.

## Fortune 500 Project Workflow

### Intra-company (single company)

```bash
curl -X POST http://localhost:8131/api/pitch \
  -d '{"pitch": "Build a SaaS invoicing tool", "model": "gpt-5.2"}'
```

Flow: `run_project()` in `base_app.py` -> AI breaks pitch into tasks -> all tasks execute in parallel -> results synthesized.

### Cross-company (holding orchestrates all subsidiaries)

```bash
# CEO pitches to holding
curl -X POST http://localhost:8130/api/pitch \
  -d '{"pitch": "Build an AI invoice SaaS"}'

# Check task breakdown
curl http://localhost:8130/api/projects/{id}

# Execute across all companies in parallel
curl -X POST http://localhost:8130/api/projects/{id}/execute

# Combine all results
curl -X POST http://localhost:8130/api/projects/{id}/integrate
```

Flow: `orchestrator.py` -> AI generates outline -> `SKILL_ROUTING` assigns tasks to companies -> each company's `/api/execute-task` runs a focused roundtable -> results synthesized.

When a roundtable synthesis includes `[ACTION:project] description [/ACTION]`, project mode auto-triggers.

### Skill routing (orchestrator.py)

`SKILL_ROUTING` maps 35 skills to companies + participants:
- `frontend`, `backend`, `architecture`, `security`, `devops`, `ai_engineering` -> MeshTech
- `marketing`, `content_creation`, `seo`, `branding`, `social_media` -> MeshMedia
- `finance`, `risk_analysis`, `data_analysis`, `pricing`, `accounting` -> MeshCapital
- `market_research`, `sales`, `ideation`, `competitive_analysis`, `go_to_market` -> MeshVentures
- `strategy`, `legal`, `tax`, `governance`, `executive_review` -> Holding

## Company Roundtable System

Each company has: `participants.json` (team roster with model IDs + personas), `company_config.json` (name, budget limits), optional `presets.json`, `model_config.json`.

| Company | Port | Domain | Key Participants |
|---------|------|--------|-----------------|
| **Holding** | 8130 | Strategy, legal, governance | CEO, CFO, COO, Chief of Staff, Business Attorney, Tax Attorney |
| **MeshTech** | 8131 | Software engineering | CTO, Tech Lead, Full-Stack Dev, Backend Dev, AI Engineer, DevOps, QA |
| **MeshMedia** | 8132 | Content, marketing | Creative Director, Social Media Manager, Copywriter, SEO |
| **MeshCapital** | 8133 | Finance, investments | Wealth Optimizer, Risk Manager, Accountant, Portfolio Manager |
| **MeshVentures** | 8134 | New business, R&D | Brainstormer, Devil's Advocate, Market Researcher, Sales Director |

Every company exposes (from `base_app.py`): `POST /api/pitch`, `POST /api/execute-task`, `GET /api/active-projects`, `GET /api/participants`, `GET /api/dashboard`, `POST /api/send-to/{company}`. Holding adds orchestration endpoints: `/api/projects`, `/api/projects/{id}/execute`, `/api/projects/{id}/integrate`, `/api/skills`.

## Extending the System

**Adding a new company**: Create `companies/<name>/main.py` using `create_company_app()`, add `participants.json` and `company_config.json`, add service to `docker-compose.yml` with `COMPANY_CODE` env var, update `COMPANY_URLS` in `orchestrator.py`.

**Adding a new proxy**: Create `proxies/<name>/{main.py,requirements.txt}`, add to `docker-compose.yml`, add `MODEL_TO_PROXY` entries in `router/routing.py`.

**Adding R730 models**: Pull model on R730 (`ssh r730@192.168.50.179`), add exact tag to `R730_MODELS` set in `routing.py`, add alias in `MODEL_ALIASES` if `:latest` tag differs.

## Multi-Agent Coordination

`collab-chat/contracts.py` defines shared contracts for multi-agent parallel development (Claude Code, Codex, OpenCode in separate worktrees under `.worktrees/`). Includes WebSocket/REST schemas, task schemas, team presets, and `FILE_OWNERSHIP` rules. Do not modify unilaterally.

## Key Ports

| Service | Port | Container |
|---------|------|-----------|
| AI Router | 8110 | ai-mesh-router |
| Claude Proxy | 8100 | ai-mesh-claude-proxy |
| ChatGPT Proxy | 8101 | ai-mesh-chatgpt-proxy |
| Grok Proxy | 8102 | ai-mesh-grok-proxy |
| Gemini Proxy | 8103 | ai-mesh-gemini-proxy |
| DeepSeek Proxy | 8105 | ai-mesh-deepseek-proxy |
| Claude Code | 8104 | ai-mesh-claude-code |
| CrewAI Agents | 8120 | ai-mesh-crewai |
| Holding Board | 8130 | ai-mesh-holding-board |
| MeshTech | 8131 | ai-mesh-meshtech |
| MeshMedia | 8132 | ai-mesh-meshmedia |
| MeshCapital | 8133 | ai-mesh-meshcapital |
| MeshVentures | 8134 | ai-mesh-meshventures |
| MeshCorp HQ | 8160 | ai-mesh-meshcorp-hq |
| Collab Chat (legacy) | 8139 | ai-mesh-collab-chat |
| Redis | 6379 | ai-mesh-redis |
| n8n | 5678 | ai-mesh-n8n |

## Networks

- `ai-mesh` — Internal bridge for all ai-mesh services
- `ai-stack_default` — External bridge to ai-stack (PostgreSQL `prompt-template-db` at 5432)

## Database

Container `prompt-template-db` from ai-stack. Default DB is `templates`, NOT `admin`. Always specify `-d ai_mesh`. `companies/shared/db.py` auto-creates all tables on first connect. `router/init_db.sql` creates router-level tables.

## API Keys

Stored in `.env`. Required: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `XAI_API_KEY`, `GOOGLE_API_KEY`, `DEEPSEEK_API_KEY`. Optional: `STRIPE_SECRET_KEY`, `RESEND_API_KEY`, `CLOUDFLARE_API_KEY`, `POSTGRES_PASSWORD`.

---
> Source: [accumulator666/ai-roundtable](https://github.com/accumulator666/ai-roundtable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
