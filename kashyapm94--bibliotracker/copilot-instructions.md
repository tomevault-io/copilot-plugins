## bibliotracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies (including dev group)
uv sync --all-groups

# Run the development server
uv run uvicorn bibliotracker.app:app --reload

# Run all tests
uv run pytest

# Run a single test
uv run pytest tests/test_app.py::test_read_root

# Lint
uv run ruff check .

# Format
uv run ruff format .

# Run database migrations
uv run alembic upgrade head

# Create a new migration
uv run alembic revision --autogenerate -m "description"

# Production startup (runs migrations then starts server)
uv run alembic upgrade head && uv run uvicorn bibliotracker.app:app --host 0.0.0.0 --port $PORT
```

## Architecture

Bibliotracker is a FastAPI web app for tracking a to-read list. Books are searched via the Google Books API, then enriched with metadata (description, region, subjects, fiction/non-fiction) by querying Claude (`claude-opus-4-6`) at add-time.

### Request flow for adding a book

1. Frontend searches → `GET /api/search` → `BookLookupService.search_books()` → Google Books API (English-only filtering)
2. User selects a book → `POST /api/add` → `BookLookupService.get_book_metadata()` → Claude API (`BookAI.get_book_details()`) → `PostgresClient.add_book()`

### Key modules

- `bibliotracker/app.py` — All FastAPI routes and admin auth middleware
- `bibliotracker/config.py` — `Config` class reads env vars at module import time (class-level attributes, not instance)
- `bibliotracker/ai.py` — `BookAI` wraps the Anthropic API (`claude-opus-4-6`), returns structured JSON parsed from AI responses
- `bibliotracker/books/google_books.py` — `GoogleBooksClient` uses `httpx` to call the Google Books API
- `bibliotracker/books/service.py` — `BookLookupService` composes `GoogleBooksClient` + `BookAI`
- `bibliotracker/storage/models.py` — SQLAlchemy `Book` model (single table: `books`)
- `bibliotracker/storage/client.py` — `PostgresClient` handles all DB operations using **synchronous** SQLAlchemy
- `bibliotracker/static/` — Vanilla JS/HTML/CSS frontend served by FastAPI (`index.html`, `stats.html`, `script.js`, `style.css`)
- `alembic/` — Migration scripts

### Frontend behaviour

- Dark theme with colorful book cards (per-card gradient via `nth-child` rules), glassmorphism footer (`backdrop-filter: blur`), gradient header text with shimmer animation, and rainbow top stripe.
- Filter bar on the main page lets users filter books by Fiction / Non-Fiction / Owned. Filters are applied server-side via query params on `GET /api/toread`.
- When adding a book the confirmation modal stays open with a "Please wait..." button state until the AI + DB round-trip completes (can take several seconds).
- Escape key closes any open modal or dropdown.
- The search dropdown shows "Searching..." immediately while the API call is in flight.
- Google Fonts used: Lora (book titles), Playfair Display (headings), Outfit (body).

### Important data model details

- `Book.subjects` is stored as a comma-separated string (max 5 subjects), not an array
- `Book.is_fiction` is stored as a plain string: `"Fiction"`, `"Non-Fiction"`, or `None`
- `Book.is_owned` is a boolean, defaults to `False`
- Duplicate prevention is case-insensitive title matching (`ilike`)

### Admin authentication

Admin-only endpoints (`POST /api/add` with `is_owned`, `PATCH /api/books/{id}`, `DELETE /api/books/{id}`, `POST /api/verify-admin`) check the `X-Admin-Password` HTTP header against the `ADMIN_PASSWORD` env var.

### Database

Uses synchronous SQLAlchemy with `postgresql+psycopg` (psycopg3). The engine is configured with `pool_pre_ping=True`, `pool_size=5`, `max_overflow=10`. Schema is auto-created via `Base.metadata.create_all()` on startup; Alembic handles incremental migrations.

`get_all_books` and `get_total_count` accept optional `filter_fiction: str | None` and `filter_owned: bool | None` parameters that add `WHERE` clauses to their queries.

### Testing

Tests mock both `PostgresClient` (patched globally in `conftest.py` before app import) and `book_service` (patched per-test via `mock_book_service_for_app` fixture). No real database or AI API calls are made in tests. The `client` fixture composes both mocks and returns a `TestClient`.

### Required environment variables

```
POSTGRES_HOST, POSTGRES_PORT, POSTGRES_DB, POSTGRES_USERNAME, POSTGRES_PASSWORD
ANTHROPIC_API_KEY
ADMIN_PASSWORD
GOOGLE_BOOKS_API_KEY   # optional
REFERER_URL            # optional, defaults to http://127.0.0.1:8000/
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kashyapm94)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kashyapm94)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
