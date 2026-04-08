## case-agent

> This repository contains the Minnesota Conciliation Court Case Agent, a FastAPI

# AGENTS.md

## Purpose
This repository contains the Minnesota Conciliation Court Case Agent, a FastAPI
backend with a React + TypeScript frontend. Use this guide for common commands,
structure, and project conventions when making changes.

## Repository structure
- `backend/` FastAPI app, agents, database models/migrations, documents, memory.
- `frontend/` React + Vite client with API and WebSocket services.
- `docs/` Manual testing checklists (auth and WebSocket).
- `README.md` High-level architecture and setup notes.

## Environment setup
1. Copy environment files:
   - `cp .env.example .env`
   - `cp backend/.env.example backend/.env` (if you need backend-specific vars)
2. Update variables for database and API keys (see root `README.md`).

## Development commands
### Backend
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

## Tests and checks
### Backend tests
```bash
cd backend
pytest
```

### Frontend lint
```bash
cd frontend
npm run lint
```

### Manual testing checklists
- `docs/TESTING_AUTH.md`
- `docs/TESTING_WEBSOCKET.md`

## Project conventions
- API routes are prefixed with `/api` when accessed from the frontend (see
  `frontend/vite.config.ts` for the proxy configuration).
- Authentication uses FastAPI-Users; protected routes expect
  `Authorization: Bearer <token>`.
- Generated court documents are stored in the database and PDFs are generated
  via the documents API (see root `README.md` for the workflow).
- Database schema changes should include an Alembic migration under
  `backend/database/migrations/versions/`.
# AGENTS.md

## Project summary
Minnesota Conciliation Court Case Agent is a FastAPI backend with a React
frontend that orchestrates multiple agents to prepare conciliation court cases,
stores generated documents, and supports real-time agent monitoring.

## Repository layout
- `backend/`: FastAPI app, agents, rules, memory, documents, auth, tests
- `frontend/`: React + TypeScript UI (Vite, Tailwind)
- `docs/`: manual testing checklists
- `.env.example`: shared environment variables template

## Environment setup
- Python 3.11+ and Node 20.19+ (or 22.12+) are required.
- Optional: use `.nvmrc` to align your local Node version with the frontend tooling.
- Copy `.env.example` to `.env` and fill in required values:
  - `DATABASE_URL`, `OPENAI_API_KEY`, `TAVILY_API_KEY`, `SECRET_KEY`,
    `FRONTEND_URL`, `ENVIRONMENT`.
- PostgreSQL 15+ with pgvector is expected for local development.

## Common commands
### Backend
- Install: `cd backend && python -m venv venv && pip install -r requirements.txt`
- Run: `uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000`
- Tests: `cd backend && pytest`

### Frontend
- Install: `cd frontend && npm install`
- Run: `npm run dev`
- Lint: `npm run lint`
- Build: `npm run build`

## Key behavior notes
- Backend APIs are proxied under `/api` from Vite; health check is `/health`.
- Auth uses JWT (FastAPI-Users). See `docs/TESTING_AUTH.md` for manual checks.
- WebSocket monitoring requires JWT in the query string; see
  `docs/TESTING_WEBSOCKET.md`.
- Generated documents are stored as text and can be converted to PDFs via the
  documents API (see root `README.md` for the workflow).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LalaIAm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LalaIAm)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
