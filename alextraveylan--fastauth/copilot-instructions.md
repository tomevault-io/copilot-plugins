## fastauth

> Provides JWT-based auth (access + refresh tokens), Google OAuth2, and user management.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Authentication microservice built with Python 3.13, FastAPI, and async PostgreSQL.
Provides JWT-based auth (access + refresh tokens), Google OAuth2, and user management.

## Commands

- Install dependencies: `uv sync`
- Run server: `uv run uvicorn fastauth.main:app --reload`
- Lint: `uv run ruff check . --fix`
- Format: `uv run ruff format .`
- Type check: `uv run ty check .`
- Test: `uv run pytest`
- Docker: `docker compose up --build`

## Architecture

Layered architecture with repository pattern:
- `fastauth/routers/` - FastAPI route handlers (auth, google_auth)
- `fastauth/services/` - Business logic (AuthService, OAuth2Service)
- `fastauth/db/` - Database layer (async SQLAlchemy, generic Repository[T])
- `fastauth/models/` - SQLModel entities (User, Token) and Pydantic schemas
- `fastauth/common/` - Settings (pydantic-settings) and custom exceptions

All database operations are async (asyncpg + SQLAlchemy async sessions).
FastAPI's `Depends()` handles dependency injection for sessions and auth.

## Code Style Rules

### Comments
- Code must be self-documenting. No comments except linter directives (`# noqa`, `# type: ignore`).
- If you need to explain code, extract it into a well-named function instead.

### Python
- Imports always at module top, never inside functions.
- Use native Python types (`list`, `dict`, `tuple`, `| None`) instead of `typing` equivalents.
- Multi-line parameters: if more than 2 params (excluding `self`), each on its own line with trailing comma.
- Prefer pure functions when possible.

### Tests (`**/test_*.py`)
- All test methods must have `-> None` return type annotation.
- Structure every test with `# Given`, `# When`, `# Then` comments.
- Use `Faker` (`fake = Faker()` at module level) for generating test data instead of hardcoded strings.
- Use `pytest.mark.parametrize` for exhaustive value coverage.

## Validation

All checks must pass before committing:
```bash
uv run ruff check . --fix
uv run ruff format .
uv run ty check .
uv run pytest
```

---
> Source: [AlexTraveylan/fastauth](https://github.com/AlexTraveylan/fastauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
