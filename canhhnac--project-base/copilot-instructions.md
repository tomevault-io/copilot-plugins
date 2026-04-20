## project-base

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Flask web application with user authentication using PostgreSQL, SQLAlchemy ORM, and Alembic migrations. Runs in Docker with separate containers for the database, migrations, and web application.

## Architecture

**Database Layer (`database.py`)**
- SQLAlchemy engine configured via `DATABASE_URL` environment variable
- Default connection: `postgresql://postgres:password@db:5432/project_db`
- `SessionLocal` factory for creating database sessions
- `get_db()` generator for Flask dependency injection (yields session, ensures cleanup)

**Models (`models.py`)**
- `User` model extends both SQLAlchemy `Base` and Flask-Login's `UserMixin`
- Password handling via Werkzeug's `generate_password_hash()` and `check_password_hash()`
- Methods: `set_password()`, `check_password()`

**Application (`main.py`)**
- Flask app with Flask-Login integration
- Session management pattern: create `SessionLocal()`, perform query, close session
- Routes: index, login, register, logout, users (list), create_user, profile
- Login required via `@login_required` decorator for protected routes

**Database Migrations (Alembic)**
- Configuration in `alembic/env.py` imports `Base` from `database.py`
- Metadata configured as `target_metadata = Base.metadata`
- Migration files in `alembic/versions/`

**Docker Architecture**
- Three-service setup: `db`, `migrations`, `web`
- Service dependencies: `web` depends on `migrations` (completed successfully), `migrations` depends on `db` (healthy)
- All services communicate via `app-network` bridge network
- Web application volume-mounted for development (`./:/app/`)

## Development Commands

### Local Development (with Docker)
```bash
# Start all services (database, run migrations, start web app)
docker-compose up

# Rebuild and start
docker-compose up --build

# Stop all services
docker-compose down
```

### Database Migrations

```bash
# Create a new migration (inside Docker)
docker-compose exec web uv run alembic revision --autogenerate -m "description"

# Run migrations manually
docker-compose exec web uv run alembic upgrade head

# Rollback one migration
docker-compose exec web uv run alembic downgrade -1

# View migration history
docker-compose exec web uv run alembic history
```

### Running Python Commands

```bash
# Execute Python in the web container
docker-compose exec web uv run python

# Run the Flask app manually
docker-compose exec web uv run python main.py
```

### Package Management (uv)

This project uses `uv` for Python package management.

```bash
# Add a new dependency
uv add <package-name>

# Sync dependencies from lock file
uv sync --frozen

# Run a command in the uv environment
uv run <command>
```

## Important Patterns

**Database Session Management**
Always follow this pattern when accessing the database:
```python
db = SessionLocal()
try:
    # perform database operations
    db.commit()  # if writing
finally:
    db.close()
```

**Adding New Models**
1. Define model in `models.py` inheriting from `Base`
2. Import model in `alembic/env.py` if not auto-discovered
3. Generate migration: `docker-compose exec web uv run alembic revision --autogenerate -m "add model_name"`
4. Review generated migration file
5. Apply: `docker-compose exec web uv run alembic upgrade head`

**Environment Variables**
- `DATABASE_URL`: PostgreSQL connection string
- `FLASK_APP`: Entry point (main.py)
- `FLASK_ENV`: development/production
- `DB_HOST`, `DB_PORT`: Used by migrations health check script

## Configuration Notes

- Flask secret key hardcoded in `main.py:7` - change for production
- PostgreSQL credentials in `docker-compose.yml` - use secrets for production
- App runs on port 8000 (mapped to host:8000)
- Debug mode enabled via `app.run(debug=True)` in `main.py:174`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CanhhnaC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
