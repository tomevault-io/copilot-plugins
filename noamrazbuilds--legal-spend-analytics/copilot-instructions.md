## legal-spend-analytics

> Legal spend and matter analytics dashboard (Project #10 in the Legal Quant portfolio). Tracks legal department workload, matter types, cycle times, outside counsel spend, and team utilization. See `PROJECT.md` for full requirements and `PORTFOLIO_STANDARDS.md` for cross-portfolio conventions.

# CLAUDE.md — Legal Matter & Spend Analytics Dashboard

## Project Overview

Legal spend and matter analytics dashboard (Project #10 in the Legal Quant portfolio). Tracks legal department workload, matter types, cycle times, outside counsel spend, and team utilization. See `PROJECT.md` for full requirements and `PORTFOLIO_STANDARDS.md` for cross-portfolio conventions.

## Architecture

**API-first pattern:** FastAPI backend handles all business logic. Streamlit is a thin HTTP client.

```
lsa_app/          — FastAPI backend (API, models, services, data import)
lsa_frontend/     — Streamlit frontend (calls API via HTTP, never touches DB directly)
data/sample/      — Synthetic sample data (CSV) for immediate testing
tests/            — pytest test suite
```

## Package Naming

This project uses prefixed package names to avoid collisions across portfolio projects:
- `lsa_app/` (not `app/`)
- `lsa_frontend/` (not `frontend/`)
- Backend entry: `uvicorn lsa_app.main:app`
- Frontend entry: `streamlit run lsa_frontend/app.py`

See `PACKAGE_NAMING.md` for rationale.

## Tech Stack

- Python 3.11+, FastAPI, Uvicorn
- SQLAlchemy 2.0 (mapped_column, DeclarativeBase) with SQLite (prototype)
- Pydantic v2 for request/response validation
- Streamlit for frontend
- Plotly for interactive charts
- pytest for testing

## Coding Conventions

- Type hints on all function signatures
- Pydantic models for all API request/response schemas
- SQLAlchemy 2.0 style (DeclarativeBase, mapped_column)
- FastAPI dependency injection for DB sessions
- Keep modules small and focused
- Minimum complexity for the current task — don't over-engineer

## Security

- All user-uploaded CSV data must be validated and sanitized before processing
- Never expose raw database errors to API consumers — use structured error responses
- Parameterized queries only (SQLAlchemy ORM handles this, but never use raw string interpolation)
- File upload size limits enforced at the API layer
- No sensitive data in logs or error messages
- CORS configured explicitly (not wildcard in production)
- Input validation on all API endpoints via Pydantic

## Data

- `data/sample/` contains synthetic CSV files for demo/testing — never real company data
- `data/real/` is gitignored — for local use only
- Database files (`*.db`) are gitignored

## Dependencies

Core `[project.dependencies]` = everything needed at runtime (FastAPI, Streamlit, SQLAlchemy, Plotly, etc.).
Dev `[project.optional-dependencies]` = testing/linting only (pytest, ruff).

Rule: if Docker would crash without it, it's a core dependency.

## Deployment

Three-tier access:
1. **Hosted** — Railway deployment (URL)
2. **Local** — `./start.sh` or `start.bat` (one command)
3. **Docker** — `docker compose up`

## Commands

```bash
# Run API server
uvicorn lsa_app.main:app --reload --port 8000

# Run frontend
streamlit run lsa_frontend/app.py --server.port 8501

# Run tests
pytest

# Run both (use start.sh for proper process management)
./start.sh
```

## License

MIT — Copyright (c) 2026 Noam Raz and Pleasant Secret Labs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noamrazbuilds) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
