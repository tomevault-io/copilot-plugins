## scout

> Scout is an agentic sales intelligence platform for VAR/SI (Value Added Reseller / Systems Integrator) sales teams. Given a company name, industry, and a vague project/initiative, Scout autonomously researches the open internet using a multi-agent system to build a living, interactive intelligence profile — everything a seller needs for effective conversations.

# CLAUDE.md — Project Scout

## Project Overview

Scout is an agentic sales intelligence platform for VAR/SI (Value Added Reseller / Systems Integrator) sales teams. Given a company name, industry, and a vague project/initiative, Scout autonomously researches the open internet using a multi-agent system to build a living, interactive intelligence profile — everything a seller needs for effective conversations.

## Architecture

### Multi-Agent System

Scout uses 4 specialized AI agents running in a loop:

1. **Prime Agent** — Strategist. Plans research, delegates to sub-agents, reviews results, decides next cycle or stop. Never calls tools directly. Uses Sonnet 4 with extended thinking.
2. **Research Sub-Agents** (up to 5 in parallel) — Focused researchers. Each gets a single assignment, executes tool calls (search, scrape, etc.), returns structured findings. Stateless and independent. Uses Sonnet 4 without extended thinking.
3. **Synthesis Agent** — Analyst. Merges raw findings from all sub-agents, deduplicates, resolves contradictions, categorizes, assesses confidence. Uses Sonnet 4 with extended thinking.
4. **Presentation Agent** — Storyteller. Formats structured intelligence into dashboard-ready content with summaries, insights, and portfolio mapping. Uses Sonnet 4 without extended thinking.

The cycle: Prime plans → Sub-agents research in parallel → Synthesis merges → Presentation formats → Dashboard updates → Prime reviews and plans next cycle OR stops.

### Tech Stack

- **Backend**: Python 3.12+ / FastAPI (async)
- **Frontend**: Next.js 14+ / React / TypeScript / Tailwind CSS
- **Database**: PostgreSQL with JSONB columns for flexible findings storage
- **AI**: Anthropic Claude API (claude-sonnet-4-5-20250514) via `anthropic` Python SDK
- **Web Search**: Brave Search API
- **Web Scraping**: httpx + BeautifulSoup4 (Playwright for JS-heavy pages as fallback)
- **SEC Data**: SEC EDGAR API (free, no key)
- **Real-time**: Server-Sent Events (SSE) for dashboard updates
- **Auth**: Email/password + JWT (v0.1)
- **Deployment**: Docker Compose (dev), GCP Cloud Run + Cloud SQL for PostgreSQL + Vercel (prod)
- **Cloud Storage**: Google Cloud Storage (GCS) for raw scraped content and documents
- **Secrets**: GCP Secret Manager for API keys and credentials
- **Logging**: Google Cloud Logging (via Cloud Run integration)
- **Container Registry**: Google Artifact Registry

### Project Structure

```
scout/
├── backend/
│   ├── app/
│   │   ├── main.py                    # FastAPI app entry point and route registration
│   │   ├── config.py                  # Pydantic Settings — env vars, API keys
│   │   ├── models/                    # Pydantic models (request/response schemas)
│   │   │   ├── company.py
│   │   │   ├── initiative.py
│   │   │   ├── research.py
│   │   │   └── user.py
│   │   ├── db/                        # Database layer
│   │   │   ├── database.py            # Async SQLAlchemy engine + session
│   │   │   ├── tables.py             # SQLAlchemy ORM table definitions
│   │   │   └── queries.py            # Reusable query functions
│   │   ├── agents/                    # Multi-agent research engine
│   │   │   ├── orchestrator.py        # Top-level cycle loop manager
│   │   │   ├── prime_agent.py         # Strategic planning agent
│   │   │   ├── research_agent.py      # Focused research worker
│   │   │   ├── synthesis_agent.py     # Findings merger and analyzer
│   │   │   ├── presentation_agent.py  # Dashboard content formatter
│   │   │   ├── prompts/              # System prompt strings per agent
│   │   │   │   ├── prime.py
│   │   │   │   ├── research.py
│   │   │   │   ├── synthesis.py
│   │   │   │   └── presentation.py
│   │   │   └── tools/                # Tool functions for research sub-agents
│   │   │       ├── base.py           # Base tool interface + Anthropic tool schema helpers
│   │   │       ├── web_search.py     # Brave Search API wrapper
│   │   │       ├── web_scrape.py     # httpx + BeautifulSoup scraper
│   │   │       ├── sec_filings.py    # SEC EDGAR API client
│   │   │       ├── job_postings.py   # Career page scraper
│   │   │       └── news_search.py    # Brave News search wrapper
│   │   ├── routers/                  # FastAPI route handlers
│   │   │   ├── research.py           # Research session endpoints
│   │   │   ├── companies.py          # Company profile endpoints
│   │   │   ├── portfolio.py          # Portfolio management endpoints
│   │   │   └── auth.py              # Authentication endpoints
│   │   ├── services/                 # Business logic layer
│   │   │   ├── research_service.py
│   │   │   ├── company_service.py
│   │   │   └── portfolio_service.py
│   │   └── streams/                  # SSE streaming
│   │       └── events.py            # Event types and SSE helpers
│   ├── tests/
│   │   ├── agents/
│   │   ├── tools/
│   │   ├── routers/
│   │   └── conftest.py
│   ├── alembic/                      # Database migrations
│   │   └── versions/
│   ├── alembic.ini
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── app/                      # Next.js app router pages
│   │   │   ├── page.tsx              # Home / new research
│   │   │   ├── research/[id]/page.tsx # Research session view
│   │   │   ├── companies/[id]/page.tsx # Company profile
│   │   │   ├── portfolio/page.tsx    # Portfolio management
│   │   │   ├── login/page.tsx
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── ResearchInput.tsx
│   │   │   ├── ResearchStatus.tsx
│   │   │   ├── Dashboard.tsx
│   │   │   ├── FindingsCard.tsx
│   │   │   ├── PeoplePanel.tsx
│   │   │   ├── InitiativePanel.tsx
│   │   │   ├── TechPanel.tsx
│   │   │   ├── CompetitivePanel.tsx
│   │   │   ├── PortfolioPanel.tsx
│   │   │   ├── FollowUpInput.tsx
│   │   │   └── DiscoveredInitiatives.tsx
│   │   ├── hooks/
│   │   │   ├── useResearchStream.ts  # SSE connection and event handling
│   │   │   └── useApi.ts            # API client hook
│   │   ├── lib/
│   │   │   ├── api.ts               # Typed API client
│   │   │   └── types.ts             # Shared TypeScript types
│   │   └── contexts/
│   │       └── AuthContext.tsx
│   ├── package.json
│   ├── tsconfig.json
│   ├── tailwind.config.js
│   └── next.config.js
├── docker-compose.yml                # Local dev: backend + postgres + frontend
├── CLAUDE.md                         # This file
└── README.md
```

## Coding Conventions

### Python (Backend)

- **Python 3.12+** — use modern syntax (type unions with `|`, etc.)
- **Async everywhere** — all FastAPI routes, database queries, agent calls, and tool functions must be `async def`
- **Type hints on all functions** — parameters and return types. Use Pydantic models for complex types.
- **Pydantic v2** for all data models (request schemas, response schemas, internal models)
- **SQLAlchemy 2.0** async style with `asyncpg` driver
- **No bare exceptions** — always catch specific exception types
- **Logging** — use `structlog` for structured JSON logging. Every agent action, tool call, and cycle event should be logged.
- **Environment variables** via Pydantic `Settings` class in `config.py` — never hardcode API keys
- **Docstrings** on all public functions and classes (Google style)
- **Tests** with `pytest` + `pytest-asyncio`. Mock external API calls (Anthropic, Brave, httpx).

### TypeScript (Frontend)

- **TypeScript strict mode** — no `any` types
- **Functional components only** — no class components
- **Named exports** — no default exports except for Next.js pages
- **Tailwind CSS** — no separate CSS files, no CSS-in-JS
- **Server components by default** — use `'use client'` only when needed (event handlers, hooks, SSE)
- **Types** — define shared types in `lib/types.ts`, keep them in sync with backend Pydantic models

### Agent Prompts

- System prompts live in `agents/prompts/` as Python string constants
- Each prompt file exports a function that accepts dynamic context (company name, prior findings, etc.) and returns the complete system prompt
- Prompts should be iterated on frequently — they are the most important lever for output quality
- Keep prompts focused: each agent role has ONE job

### Tool Implementation

- Each tool in `agents/tools/` must:
  1. Define an `TOOL_SCHEMA` dict matching the Anthropic tool use JSON schema
  2. Implement an `async def execute(**kwargs)` function
  3. Return structured data (dict), never raw HTML or unprocessed text
  4. Handle errors gracefully — return error information, don't raise exceptions that kill the sub-agent
  5. Implement reasonable timeouts (10s for search, 15s for scraping)

### Database

- Use Alembic for all schema migrations — never modify tables manually
- JSONB columns use Pydantic models for validation before insert
- All queries go through `db/queries.py` — routers and services never build raw SQL
- Foreign key constraints enforced at the database level
- Soft deletes where appropriate (add `deleted_at` column)

### SSE Events

- All SSE events follow the format: `event: {event_type}\ndata: {json_payload}\n\n`
- Event types are defined as an enum in `streams/events.py`
- Every event includes a `timestamp` and `session_id`
- Key event types:
  - `cycle_started` — new research cycle beginning
  - `subagent_started` — individual sub-agent dispatched (includes topic)
  - `subagent_completed` — sub-agent finished (includes finding count)
  - `subagent_stopped` — user stopped a sub-agent
  - `synthesis_complete` — synthesis agent finished
  - `findings_updated` — new dashboard content available
  - `initiative_discovered` — agent found a new initiative to surface
  - `research_complete` — entire session finished
  - `error` — something went wrong

### API Design

- RESTful endpoints under `/api/v1/`
- All responses use consistent envelope: `{ "data": ..., "error": null }` or `{ "data": null, "error": { "message": "...", "code": "..." } }`
- Pagination with `?offset=0&limit=20` for list endpoints
- SSE streams are separate endpoints, not mixed with REST

## Key Design Decisions

1. **Custom agent orchestration** (no LangGraph/CrewAI) — the multi-agent loop is simple enough to implement directly with asyncio and the Anthropic SDK. Avoids framework lock-in and abstraction overhead.
2. **Sub-agent cap of 5 per cycle** — controls API costs and avoids overwhelming data sources.
3. **Research sub-agents are stateless** — they receive an assignment, execute, return results. No inter-agent communication. The Prime Agent coordinates everything.
4. **JSONB for findings** — research findings have variable structure by category. JSONB allows schema evolution without migrations.
5. **SSE over WebSockets** — one-directional updates (server → client) are all we need. SSE is simpler to implement and debug.
6. **Brave Search over Google** — generous free tier, simple API, no OAuth complexity.
7. **Extended thinking only where needed** — Prime Agent and Synthesis Agent use it (deep reasoning). Research and Presentation agents don't (speed matters more).

## Common Commands

```bash
# Start local dev environment
docker-compose up -d

# Run backend only
cd backend && uvicorn app.main:app --reload --port 8000

# Run frontend only
cd frontend && npm run dev

# Run backend tests
cd backend && pytest -v

# Run database migrations
cd backend && alembic upgrade head

# Create new migration
cd backend && alembic revision --autogenerate -m "description"
```

## Environment Variables

```
# Backend (.env)
ANTHROPIC_API_KEY=sk-ant-...
BRAVE_SEARCH_API_KEY=BSA...
DATABASE_URL=postgresql+asyncpg://scout:scout@localhost:5432/scout
JWT_SECRET=<random-string>
ENVIRONMENT=development

# Frontend (.env.local)
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
```

## Important Context

- This is a v0.1 — optimize for speed of iteration, not perfection
- Primary users are enterprise sales professionals, not developers — the UI should be clean and readable
- The agentic research loop is the core product — spend the most effort on prompt engineering and agent orchestration quality
- Portfolio mapping (matching findings to the user's vendor partnerships) is a key differentiator
- The system should gracefully handle tool failures — a failed scrape shouldn't crash a research session
- Research sub-agents should have a tool call budget of 8–10 calls per assignment

---
> Source: [aquaregiaswarm-blip/scout](https://github.com/aquaregiaswarm-blip/scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
