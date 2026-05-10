## bible-tracker

> Bible Reading Tracker MVP — a single-user API-driven app for logging Bible reading progress with a vanilla JS frontend. Built as a learning project with a 20–40 hour scope.

# CLAUDE.md

## Project Overview

Bible Reading Tracker MVP — a single-user API-driven app for logging Bible reading progress with a vanilla JS frontend. Built as a learning project with a 20–40 hour scope.

## Tech Stack

- **Backend**: FastAPI, SQLAlchemy (sync), SQLite
- **Frontend**: Vanilla HTML/CSS/JS (no framework, no build tools)
- **Packaging**: uv
- **Testing**: pytest + httpx

## Project Structure

```
bible_tracker/
├── models/          # SQLAlchemy models (book.py, reading.py)
├── schemas/         # Pydantic request/response schemas
├── routes/          # FastAPI routers (readings.py, progress.py)
├── data/            # Canon CSV seed files (P, C, J)
├── static/          # Frontend (index.html, style.css, app.js)
├── tests/
├── database.py      # Engine, SessionLocal, Base, get_db dependency
├── main.py          # FastAPI app entry point
├── seed_db.py       # Creates tables and seeds book data from CSVs
├── test_db.py       # Verifies database setup and seed data
└── test_main_routes.py  # 32 route tests
```

## Key Commands

```bash
uv run python seed_db.py          # Create tables and seed book data
uv run uvicorn main:app --reload  # Run dev server
uv run pytest test_main_routes.py -v  # Run route tests
uv run python test_db.py          # Verify database setup
```

## Database

- SQLite file: `bible_tracker.db` (gitignored)
- Test database: `test_bible_tracker.db` (created/destroyed per test)
- `get_db` is defined in `database.py` and imported by all route modules
- Three canons seeded: Protestant (P: 78 books), Catholic (C: 72), Jewish (J: 24)

## API Design

All endpoints are under `/api/`. Canon is passed as a query param defaulting to `P`.

- `POST /api/readings/?canon=P` — log a reading passage
- `GET /api/readings/?canon=P&book_id=&start_date=&end_date=` — list history
- `DELETE /api/readings/{id}` — delete a reading
- `GET /api/progress?canon=P` — per-book chapter completion, grouped by testament
- `GET /api/progress/summary?canon=P` — OT/NT/AP/total aggregate stats
- `GET /api/heatmap` — rolling 365-day reading counts

## Data Model

**Book**: id, name, canon (P/C/J), testament (OT/NT/AP), chapter_count, verse_count, book_order

**Reading**: id, book_id (FK), start_chapter, start_verse, end_chapter, end_verse, date_read, created_at

## Important Conventions

- **Chapter counting**: Readings span a range (start_chapter to end_chapter). Progress calculations expand ranges into sets of individual chapters and deduplicate across readings. Overlapping readings do not double-count.
- **Validation**: Pydantic schemas enforce positive values and chapter/verse ordering. Routes validate chapters against the book's chapter_count. Canon filtering prevents logging readings against books from the wrong canon.
- **No auth**: Single-user app, no authentication.
- **No async**: Sync SQLAlchemy deliberately chosen for MVP simplicity.
- **Heatmap**: Rolling 365-day window from today, not calendar year.

## Testing

- Tests use a separate SQLite database with dependency override on `get_db`
- Tables are created before each test and dropped after
- Test books are a subset (Genesis, Exodus, Matthew, Revelation, Tobit)
- 32 tests covering CRUD, validation, progress calculation, and heatmap

## Current Status

Phases 1–4 complete (setup, database, API, tests). Frontend phases 5–9 are next.

## What This Project Is NOT

This is a scoped-down learning exercise. The full version will use async SQLAlchemy, PostgreSQL, Docker, separate frontend/backend repos, user auth, and deploy to a VPS. Don't over-engineer this MVP.

---
> Source: [nateonmission/bible_tracker](https://github.com/nateonmission/bible_tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
