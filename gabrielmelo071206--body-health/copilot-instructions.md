## body-health

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a FastAPI-based e-commerce application (Loja Virtual) built with Python. The application follows an MVC pattern with SQLite database storage and Jinja2 templating for the frontend.

## Tech Stack

- **Backend**: FastAPI with Uvicorn ASGI server
- **Database**: SQLite with raw SQL queries
- **Frontend**: Jinja2 templates with static HTML/CSS/JS
- **Python**: 3.12+ (based on .venv structure)

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the development server
python main.py
# Server runs on http://127.0.0.1:8000 with auto-reload enabled
```

## Architecture

### Directory Structure
- `main.py` - Application entry point, FastAPI app initialization, router registration
- `model/` - Data models using Python dataclasses
- `repo/` - Repository layer for database operations (CRUD operations)
- `sql/` - SQL query definitions
- `data/` - Database utilities (connection management)
- `routes/` - API route definitions
  - `public_routes.py` - Public-facing routes
  - `admin/` - Admin panel routes for entity management
- `templates/` - Jinja2 HTML templates
- `static/` - Static assets (CSS, JS, images)
- `dados.db` - SQLite database file

### Key Architectural Patterns

1. **Repository Pattern**: Each entity has a dedicated repository file in `repo/` handling all database operations
2. **SQL Separation**: SQL queries are separated into `sql/` modules for maintainability
3. **Model-Repository-Route Structure**: Each entity follows the pattern:
   - Model definition in `model/[entity]_model.py`
   - Repository operations in `repo/[entity]_repo.py`
   - SQL queries in `sql/[entity]_sql.py`
   - Routes in `routes/admin/admin_[entity]_routes.py`

### Core Entities
- Categoria (Category)
- Produto (Product)
- Cliente (Customer)
- Usuario (User)
- Admin
- FormaPagamento (Payment Method)

### Database Initialization
Tables are created automatically on application startup via `criar_tabela()` calls in main.py. The database connection is managed through `data/util.py` using sqlite3 with Row factory for dictionary-like row access.

### Routing Structure
- Public routes serve the main store interface
- Admin routes follow the pattern `/admin/{entity}` for CRUD operations
- Templates are organized with admin templates in `templates/admin/`

## Development Notes

- The application auto-creates database tables on startup
- FastAPI's automatic reload is enabled when running via main.py
- Static files are served from the `/static` mount point
- All database operations use raw SQL queries through sqlite3

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GabrielMelo071206) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
