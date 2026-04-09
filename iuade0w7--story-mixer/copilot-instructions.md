## story-mixer

> LoreForge is a calibrated narrative generation platform. Users set vibe sliders

# LoreForge — Claude Code Context

## What This Project Does
LoreForge is a calibrated narrative generation platform. Users set vibe sliders
(Aggression, Reader Respect, Morality, Source Fidelity) and chapter settings that shape
an AI-generated long-form story. The backend runs a multi-agent pipeline: an outline agent
builds a chapter plan, a chapter writer streams each chapter, and a critic agent evaluates
and drives revision before returning the final result via SSE.

## Repository Layout
```
story-mixer/
├── frontend/          # Next.js 15 + React 19 + TypeScript + Tailwind + Radix UI
├── backend/           # FastAPI + SQLAlchemy + Alembic + LangChain
├── .venv/             # Python venv (not committed)
├── Makefile           # Top-level task runner — prefer make targets over raw commands
└── CLAUDE.md
```

## Architecture

### Frontend (`frontend/`)
- `src/app/` — Next.js App Router pages
- `src/components/` — React components (VibeController sliders, story output)
- API calls proxy to backend via Next.js rewrites (`next.config.ts`)
- Streams story events via EventSource / SSE

### Backend (`backend/app/`)
- `main.py` — FastAPI app factory
- `api/` — Route handlers (stories, health)
- `services/` — Business logic:
  - `long_form_orchestrator.py` — coordinates outline → chapter write → critic pipeline
  - `outline_agent.py` — generates chapter outline from vibe settings and brief
  - `critic_agent.py` — evaluates chapter drafts and drives revision loop
  - `provider_gateway.py` — LLM provider abstraction (Ollama/OpenAI/Anthropic/Gemini)
- `domain/` — Pydantic models / domain types
- `persistence/` — SQLAlchemy models (Alembic-managed schema)
- `config.py` — Settings loaded from env vars

### Database
- PostgreSQL, accessed async via `asyncpg` + SQLAlchemy async session
- Schema managed with **Alembic** (`backend/alembic/`)
- Default local URL: `postgresql+asyncpg://mikha:postgres@127.0.0.1:5432/loreforge`

## How to Run

### Prerequisites
```bash
make venv          # create .venv
make deps          # install Python dependencies
make db-start      # start PostgreSQL (WSL: sudo service postgresql start)
make db-ensure     # create DB user + database
make migrate       # run Alembic migrations
```

### Backend dev server
```bash
cd backend
DATABASE_URL='postgresql+asyncpg://mikha:postgres@127.0.0.1:5432/loreforge' \
  ../.venv/bin/python -m uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

### Frontend dev server
```bash
cd frontend && npm run dev   # starts on http://localhost:3000
```

### Quick smoke test (no real LLM needed)
```bash
make smoke-stream   # spins up server with USE_STUB_LLM=true, fires a curl request
```

## Testing

### Backend
```bash
make test                          # runs pytest -q (unit + integration without live providers)
make test-real-provider            # runs integration tests against real LLM provider
```

### Frontend E2E
```bash
cd frontend && npm run e2e         # Playwright headless
cd frontend && npm run e2e:headed  # with browser visible
cd frontend && npm run e2e:ui      # Playwright interactive UI
```

### Frontend lint
```bash
cd frontend && npm run lint        # next lint
```

## Key Conventions

### Python (backend)
- Python 3.12+, strict typing with Pydantic v2
- Async throughout — use `async/await` + SQLAlchemy async sessions
- New API routes go in `backend/app/api/`; business logic stays in `backend/app/services/`
- Alembic for all schema changes — never use `AUTO_CREATE_SCHEMA=true` in production
- Tests in `backend/tests/`; mark live-provider tests with `@pytest.mark.integration`

### TypeScript (frontend)
- React 19 + Next.js App Router — prefer Server Components where there's no interactivity
- Tailwind for styling; Radix UI for accessible primitives
- Keep vibe-control state in the VibeController component; lift only what needs to be shared
- Streaming: use `EventSource` / `ReadableStream` — don't buffer the full story before rendering

### Environment
- Never commit `.env` files; copy `.env.example` → `.env` and fill credentials locally
- `USE_STUB_LLM=true` for local dev without provider keys

## LLM Providers Supported
| Provider | Key env var |
|----------|-------------|
| Ollama (local) | `OLLAMA_BASE_URL` (auto-detected in WSL) |
| OpenAI | `OPENAI_API_KEY` |
| Anthropic | `ANTHROPIC_API_KEY` |
| Google Gemini | `GOOGLE_API_KEY` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IUADE0W7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IUADE0W7)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
