## tra-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TRA Treasurer Assistant — a Flask web app for Tenant and Resident Association treasurers to manage finances (income/expenditure cashbooks, petty cash, budgets, bank reconciliation, treasurer reports). Based on Hammersmith & Fulham Council TRA finance training materials.

## Commands

```bash
uv sync              # Install dependencies
uv run python app.py # Run dev server (uvicorn, port 5000, auto-reload)
```

No test suite or linter is currently configured.

## Architecture

Two Python files make up the entire backend:

- **`app.py`** — Flask routes, authentication decorator (`login_required`), Jinja2 template filters, category/fund constants, CSV parsing (`parse_bank_csv`), and the ASGI wrapper for uvicorn. All routes live in this single file, organized by section comments (Authentication, Income, Expenditure, Petty Cash, Budget, Reconciliation, Report, Settings).
- **`db.py`** — SQLite database layer. Schema creation (`init_db`), connection factory (`get_db`), and all query functions. Database lives at `data/tra.db` (auto-created, gitignored). Uses WAL mode and `sqlite3.Row` for dict-like access. Tables: `users`, `settings`, `income`, `expenditure`, `petty_cash`, `budget`, `bank_statements`.

Templates in `templates/` extend `base.html` (sidebar layout). Static assets served from `/` via `static/`.

## Key Patterns

- **DB connections**: Flask `g` object stores per-request connection via `get_conn()`, closed in `teardown_appcontext`.
- **Auth**: Session-based. Admin role checked inline (`session.get("user_role") != "admin"`), not via decorator. Default admin account (`admin`/`admin`) auto-created on startup if no users exist.
- **ASGI**: Flask WSGI app wrapped with `asgiref.WsgiToAsgi` and served by uvicorn. The ASGI entrypoint is `app:asgi_app`.
- **Categories**: `INCOME_CATEGORIES`, `EXPENDITURE_CATEGORIES`, and `FUND_TYPES` are hardcoded lists in `app.py`.
- **Currency**: Amounts stored as `REAL` in SQLite. The `parse_amount()` helper strips `£` and commas. Jinja `|currency` filter formats as `£X,XXX.XX`.
- **Settings**: Key-value pairs in `settings` table with defaults in `db.DEFAULT_SETTINGS`.
- **Bank reconciliation**: Two workflows — CSV upload (Bank Statement tab) or manual tick-off (Cashbook tab). Uploaded CSV rows are stored in `bank_statements` table and matched to income/expenditure entries. Matching a bank row auto-sets `reconciled=1` on the cashbook entry; unmatching reverses it. CSV format expected: `Date,Details,Transaction Type,In,Out` with DD/MM/YYYY dates (e.g. Metro Bank export). Parsed via `parse_bank_csv()` in `app.py`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/charrus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
