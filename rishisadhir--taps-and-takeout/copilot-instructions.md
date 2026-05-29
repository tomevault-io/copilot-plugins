## taps-and-takeout

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A minimal Flask website for Taps & Takeout, a neighborhood pub. Intentionally simple and low-key by design — the client wants a basic site with optional event listings.

- Live site: https://taps-and-takeout.onrender.com/
- Admin panel: https://taps-and-takeout.onrender.com/admin (username: `admin`)

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally
python app.py   # respects PORT, defaults to 5001 locally
# or
flask run --port 5001

# Production (via Render using Procfile)
gunicorn app:app
```

## Environment Variables

Create a `.env` file with:
```
FLASK_SECRET_KEY=...
ADMIN_PASSWORD=...
```

Both variables are required. The app now fails fast at startup if either one is missing.

## Architecture

- **`app.py`** — thin entrypoint that exposes `app` for gunicorn / Flask
- **`taps_and_takeout/app_factory.py`** — builds the Flask app, configures extensions, registers blueprints
- **`taps_and_takeout/routes/public.py`** — `/`, `/menu`, `/events`, `/contact`, `/healthz`
- **`taps_and_takeout/routes/admin.py`** — `/admin`, `/logout`, `/admin-events`, `/admin-menu`
- **`taps_and_takeout/storage.py`** — current JSON-backed content-store abstraction; intended swap point for SQLite later
- **`taps_and_takeout/validation.py`** — sanitization and length limits for admin-submitted content
- **`taps_and_takeout/logging_utils.py`** — structured logging helpers for admin actions and validation failures
- **`events.py`** — `load_events()` / `save_events()`: reads/writes `data/events.json`, converting between `datetime.date` objects and ISO strings
- **`menu_data.py`** — `load_menu()` / `save_menu()`: reads/writes `data/menu.json`; falls back to built-in default seed if file missing
- **`data/events.json`** — flat JSON array of `{title, date, description}` objects; this is the events database
- **`data/menu.json`** — JSON array of `{section, items: [{name, description}]}` objects; this is the menu database
- **`templates/`** — Jinja2 templates; admin pages extend `admin_base.html` which extends `base.html`
- **`static/`** — single `style.css` + `images/` directory
- **`tests.py`** — pytest suite (currently 50 tests); run with `pytest tests.py -v`
- **`tests_e2e.py`** — Playwright smoke tests for real browser admin event and menu flows; run with `pytest tests_e2e.py -v` after `pip install -r requirements-dev.txt` and `python -m playwright install chromium`
- **`.github/workflows/ci.yml`** — CI runs the route suite and Playwright smoke tests on push/PR

## Key Behaviors

- Events page filters out events older than yesterday (`event.date >= yesterday`) and sorts by date — both done in Python in the `/events` route, not in the template
- Admin is session-based (8-hour timeout); login at `/admin`, manage events at `/admin-events`, manage menu at `/admin-menu`; login is rate-limited to 10 attempts/minute
- Admin POST handlers validate malformed numeric indices and return `400` instead of raising `500`
- Admin inputs are sanitized and capped before they are written to disk
- Admin actions and validation failures are logged in a structured payload to stdout
- Local `python app.py` runs respect `PORT` and default to `5001`; this avoids common macOS conflicts on `5000` and matches Render's runtime model
- `/healthz` returns a small JSON payload suitable for uptime or deploy checks
- No actual database — data is stored as JSON on disk, which means it resets on Render deploys unless a persistent disk is configured

---
> Source: [RishiSadhir/TAPS-AND-TAKEOUT](https://github.com/RishiSadhir/TAPS-AND-TAKEOUT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
