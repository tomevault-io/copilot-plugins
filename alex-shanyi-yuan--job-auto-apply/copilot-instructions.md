## job-auto-apply

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Start all services
docker-compose up --build

# Frontend only (faster iteration, outside Docker)
cd frontend && npm run dev       # http://localhost:3000
cd frontend && npm run lint      # ESLint

# Run database migrations
docker-compose exec tailor alembic upgrade head

# Create a new migration
docker-compose exec tailor alembic revision --autogenerate -m "description"

# View service logs
docker-compose logs -f tailor
docker-compose logs -f frontend

# Backend (resume-tailor) without Docker
cd backend/services/resume-tailor
pip install -r requirements.txt
uvicorn server:app --reload --port 8000
```

> Note: PDF compilation (`latex_compiler.py`) requires TeX Live, which is only available in the Docker container. Run `tailor` in Docker when testing resume tailoring end-to-end.

## Architecture

AutoCareer is a self-hosted job automation platform with 4 Docker Compose services:

| Service    | Port | Stack                                                   | Role                           |
| ---------- | ---- | ------------------------------------------------------- | ------------------------------ |
| `frontend` | 3000 | Next.js 14, React 19, TypeScript, Tailwind, shadcn/ui   | Web UI                         |
| `tailor`   | 8000 | Python 3.11, FastAPI, SQLModel, Google Gemini, TeX Live | Main API + AI                  |
| `scraper`  | 8001 | Python 3.11, FastAPI, Playwright                        | Headless browser for job pages |
| `postgres` | 5432 | PostgreSQL 15                                           | Persistence                    |

**Request flow:** Frontend → `tailor` API (port 8000) → `scraper` service (port 8001) for HTML fetching → PostgreSQL or SQLite depending on `DATABASE_BACKEND`.

### Key Source Files

- `backend/services/resume-tailor/server.py` — All FastAPI endpoints (14 routes)
- `backend/services/resume-tailor/core/agents.py` — 4 AI agents (Discovery, Scoring, Parsing, Tailoring)
- `backend/services/resume-tailor/core/llm_client.py` — Gemini API client with structured output via Pydantic, retry/backoff
- `backend/services/resume-tailor/database.py` — SQLModel ORM models and backend selection: `Settings`, `JobSource`, `Job`
- `backend/services/resume-tailor/core/db_sync.py` — PostgreSQL/SQLite reconciliation and one-time migration helpers
- `backend/scripts/migrate_postgres_to_sqlite.py` — One-time export from PostgreSQL to SQLite
- `backend/services/job-scraper/main.py` — `POST /scrape` endpoint using Playwright
- `frontend/lib/api.ts` — Typed client for all backend endpoints
- `backend/services/resume-tailor/data/master.tex` — Master LaTeX resume template

### Job Status Lifecycle

```
suggested → processing → applied → interviewing → offer
                      ↘ failed                  ↘ rejected
          dismissed (user action)
```

### AI Agents

All agents call Google Gemini via `llm_client.py` using Pydantic-enforced JSON schemas:

1. **JobDiscoveryAgent** — Parses source HTML, extracts job listings, applies user filter
2. **JobScoringAgent** — Scores each discovered job 0–100 against master resume
3. **JobParsingAgent** — Extracts structured requirements from a full job description
4. **ResumeTailorAgent** — Rewrites LaTeX resume sections to match a specific job

### Suggestions Scan (Parallel Processing)

`POST /suggestions/refresh` triggers:

1. Parallel source scans (up to `MAX_CONCURRENT_SOURCES=5`)
2. Per source: scraper fetches HTML → `JobDiscoveryAgent` extracts listings → relative URLs resolved to absolute
3. Parallel scoring within each source (up to `MAX_CONCURRENT_JOBS=10` via thread pool)
4. All jobs saved as `status="suggested"` regardless of score
5. Frontend polls `GET /suggestions/status` every 2 seconds until `is_scanning=false`

### Frontend Patterns

- All API calls go through `lib/api.ts` — add new endpoints there
- State: `useState` + `useEffect` (no external state library)
- Polling uses `setInterval` in `useEffect` with cleanup on unmount
- UI uses shadcn/ui components from `components/ui/`

### Backend Patterns

```python
# Background task
background_tasks.add_task(process_application, request.url)

# DB session
with Session(engine) as session:
    job = session.exec(select(Job).where(Job.id == id)).first()
```

## Environment Variables

**Backend** (`.env` at repo root, loaded by `tailor` service):

```
GOOGLE_API_KEY=xxx                  # Required - Gemini API
DATABASE_BACKEND=hybrid             # postgres | sqlite | hybrid
SQLITE_DATABASE_URL=sqlite:///./backend/services/resume-tailor/data/autocareer.db
POSTGRES_DATABASE_URL=postgresql://user:password@postgres:5432/autocareer
DB_SYNC_ENABLED=true
SYNC_ON_BOOT=true
SYNC_ON_SHUTDOWN=true
SCRAPER_SERVICE_URL=http://scraper:8001
MASTER_RESUME_PATH=./data/master.tex
RATE_LIMIT_DELAY=0.2
MAX_CONCURRENT_SOURCES=5
MAX_CONCURRENT_JOBS=10
```

**Frontend** (`frontend/.env.local`):

```
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## Database Migrations

Migrations live in `backend/services/resume-tailor/migrations/versions/`. They run automatically on container startup via `entrypoint.sh`. When adding new columns or tables, always create a migration rather than modifying SQLModel models alone.

For hybrid mode or data portability, use `backend/scripts/migrate_postgres_to_sqlite.py` before switching the runtime backend to SQLite.

## Testing Gotchas

- Scraper is blocked by some job sites — test with different URLs
- AI responses are non-deterministic — verify output quality manually
- PDF compilation fails silently if LaTeX is malformed — check `docker-compose logs tailor`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alex-Shanyi-Yuan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Alex-Shanyi-Yuan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
