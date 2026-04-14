## juiz-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a FastMCP-based Todo application with Google Calendar and Tasks integration. The architecture follows a modular design:

- **main.py**: Entry point that defines MCP tools/endpoints and imports services
- **Service modules**: Business logic separated into `todo_service.py`, `event_service.py`, and `google_api.py`
- **models.py**: SQLAlchemy ORM models for `TodoItem`, `EventItem`, and `GoogleCredentials`
- **schemas.py**: Pydantic schemas for data validation and serialization
- **Database**: Uses PostgreSQL (remote AWS RDS) in production, SQLite fallback for development

### Key Features
- TODO management with CRUD operations
- Calendar event management  
- Google OAuth2 integration for syncing with Google Tasks and Calendar
- Database persistence with Alembic migrations
- FastMCP framework for tool/resource/prompt endpoints

## Development Commands

### Server Management
```bash
# Start the server (default port 8000)
python main.py

# Install dependencies
pip install -r requirements.txt
```

### Database Operations
```bash
# View all tables
psql $DATABASE_URL -c "\dt"

# View todos
psql $DATABASE_URL -c "SELECT * FROM todos;"

# View events  
psql $DATABASE_URL -c "SELECT * FROM events;"

# View Google credentials (sensitive)
psql $DATABASE_URL -c "SELECT user_id, created_at, updated_at FROM google_credentials;"
```

### Migration Management
```bash
# Generate new migration
alembic revision --autogenerate -m "description"

# Run migrations
alembic upgrade head

# Show current migration
alembic current
```

## Environment Setup

Required environment variables in `.env`:
- `DATABASE_URL`: PostgreSQL connection string (automatically converted from postgres:// to postgresql:// for SQLAlchemy)
- `PORT`: Server port (defaults to 8000, used by Heroku)

## Service Integration

### Google APIs
- OAuth2 flow: `start_google_oauth` → `complete_google_oauth` → `check_google_credentials`
- Services: Google Calendar v3, Google Tasks v1
- Scopes: `calendar`, `tasks`
- Credentials stored encrypted in `google_credentials` table

### Data Flow
1. Local operations (todos/events) saved to PostgreSQL
2. Optional sync to Google APIs when `sync_to_google=True`
3. Combined results returned when `include_google_*=True`
4. Error handling: Google API failures don't break local operations

## Database Schema

- **todos**: id, user_id, title, description, completed, created_at
- **events**: id, user_id, title, description, start_time, end_time, location, created_at  
- **google_credentials**: id, user_id, access_token, refresh_token, token_uri, client_id, client_secret, scopes, expiry, created_at, updated_at

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/okihara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
