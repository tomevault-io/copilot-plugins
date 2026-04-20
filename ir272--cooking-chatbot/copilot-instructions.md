## cooking-chatbot

> LLM-powered cooking & recipe Q&A application with LangGraph, FastAPI, and Next.js.

# Cooking Chatbot — Project Instructions

## Project Overview
LLM-powered cooking & recipe Q&A application with LangGraph, FastAPI, and Next.js.

## Architecture
- **Backend**: FastAPI + LangGraph StateGraph at `backend/`
- **Frontend**: Next.js 16 + Tailwind CSS at `frontend/`
- **Monorepo**: Docker Compose at root

## Key Technical Decisions
- Custom StateGraph (not create_react_agent) for classification gate
- GPT-4o-mini via LangChain ChatOpenAI
- SSE streaming (not WebSockets)
- Raw fetch + ReadableStream on frontend (no Vercel AI SDK)
- uv for Python, pnpm for Node.js

## Graph Flow
```
START → classify_query → (cooking) → agent ⇄ tools → END
                       → (off_topic) → reject_query → END
```

## Development Commands

### Backend
```bash
cd backend
uv run uvicorn app.main:app --reload --port 8001
uv run pytest -v
uv run ruff check .
uv run ruff format .
```

### Frontend
```bash
cd frontend
pnpm dev --port 3001
pnpm build
pnpm lint
```

### Docker
```bash
docker compose up --build
```

## File Conventions
- Backend Python: snake_case, type hints, Pydantic v2 models
- Frontend TypeScript: camelCase, strict mode, functional components
- Tests: `tests/test_*.py` for backend, colocated for frontend

## Environment
- `.env` at project root (never commit — in .gitignore)
- Required: `OPENAI_API_KEY`, `TAVILY_API_KEY` (optional, DuckDuckGo fallback works)

## Important Notes
- Classification node output should never be streamed to frontend
- Cookware list is hardcoded in `backend/app/tools/cookware.py`
- InMemorySaver is used for dev — swap to PostgresSaver for production
- Backend port: 8001 (8000 may be in use)
- Frontend port: 3001 (3000 may be in use)
- Graph node functions (classify_query, agent) are async — use `ainvoke` not `invoke`
- Error messages to clients are sanitized — raw exceptions are logged server-side only
- DuckDuckGo is a fallback, not additive — only used when Tavily key is absent
- Docker containers run as non-root users
- Docker Compose maps 8001:8000 (backend) and 3001:3000 (frontend)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ir272) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
