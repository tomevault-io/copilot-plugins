## buz-eta-report

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Flask web application that generates ETA (Estimated Time of Arrival) reports by fetching order data from external OData APIs (BuzManager). The app supports multiple instances (DD and CBR), handles authentication via Google OAuth, manages customer configurations, and exports reports to CSV/XLSX formats.

## Development Commands

### Setup and Installation
```bash
# Install dependencies
pip install -r requirements-dev.txt

# Initialize database
export DATABASE=/path/to/your/database.db
flask init-db

# Database backup
flask db-backup [--dir /path/to/backup]
```

### Running the Application
```bash
# Development mode (set APP_ENV=development in .env)
python app.py

# Production mode
gunicorn app:app
```

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_routes.py

# Run with verbose output
pytest -v

# Run specific test
pytest tests/test_routes.py::test_admin_page
```

### API
The `/api/v1/` REST API is implemented as a Flask Blueprint in the `api/` package.

**Authentication**: API key via `X-API-Key` header. Set `BUZ_API_KEY` environment variable.

**Endpoints**:
| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/v1/customers` | List all customers |
| GET | `/api/v1/customers/<obfuscated_id>` | Get single customer |
| POST | `/api/v1/customers` | Create customer |
| PUT | `/api/v1/customers/<obfuscated_id>` | Update customer |
| DELETE | `/api/v1/customers/<obfuscated_id>` | Delete customer |
| POST | `/api/v1/reports/<obfuscated_id>/generate` | Start async report (returns job_id) |
| GET | `/api/v1/reports/<obfuscated_id>/download?format=csv\|xlsx` | Download report |
| GET | `/api/v1/jobs/<job_id>` | Job status/progress |
| GET | `/api/v1/statuses` | List status mappings |
| POST | `/api/v1/statuses/refresh` | Sync statuses from OData |
| GET | `/api/v1/health` | Health check (no auth required) |

**Response format**: `{"data": ..., "meta": {...}}` on success, `{"error": "message", "code": "ERROR_CODE"}` on error.

**Bot workflow**: Create customer -> POST generate -> Poll GET jobs/<id> -> GET download

### Cache Prewarming
```bash
# Prewarm cache for all customers (before blackout periods)
flask prewarm-cache

# Prewarm specific instances
flask prewarm-cache --instance DD --instance CBR
```

### Maintenance
```bash
# Purge completed/failed jobs older than 30 days (dry-run by default)
flask purge-jobs
flask purge-jobs --days 7 --confirm
```

## Architecture

### Multi-Instance OData Architecture
The application connects to two separate OData instances:
- **DD (DESDR)**: Uses `BUZ_DD_USERNAME` and `BUZ_DD_PASSWORD`
- **CBR (WATSO)**: Uses `BUZ_CBR_USERNAME` and `BUZ_CBR_PASSWORD`

Each customer record in the database can have a `dd_name` and/or `cbr_name`, and the system fetches data from both instances if configured.

### Async Report Generation
Reports are generated asynchronously to handle long-running OData queries:
1. **Route** (`/<obfuscated_id>`): Creates a job in the `jobs` table and spawns a background thread
2. **Worker** (`services/eta_worker.py` or `_run_eta_report_job()`): Executes in ThreadPoolExecutor
3. **Job Service** (`services/job_service.py`): Tracks progress, logs, and completion status
4. **Polling UI** (`templates/report_loading.html`): Frontend polls `/jobs/<job_id>` endpoint
5. **Render** (`/report/<job_id>`): Serves completed report from job result

The job tracking includes:
- Progress percentage
- Log messages
- Error handling with Sentry integration
- Stall detection (STALL_TTL = 300 seconds / 5 minutes - only for truly hung workers)

### Cache Strategy (services/cache.py & services/fetcher.py)
The application uses a SQLite-backed cache to handle:
- **Blackout periods**: Falls back to cached data when live OData is unavailable (503s)
- **Timeouts**: Returns cached data on connection errors or slow responses
- **Cooldown**: After detecting a 503, avoids hammering the API for N minutes
- **Force refresh**: Can bypass cache for real-time data when OData is available

Key function: `fetch_or_cached()` in `services/fetcher.py` orchestrates this logic.

### Database Schema
The database uses SQLite with Flask's `g` object for per-request connections:
- **customers**: Maps customer names to obfuscated IDs; supports `field_type` of "Customer Name" or "Customer Group"
- **users**: OAuth user management with roles (admin/user) and active status
- **status_mapping**: Maps OData production statuses to custom display names; can be synced from live data
- **jobs**: Tracks async report generation with status, progress, logs, and results
- **cache**: Generic key/value store with timestamps for cache invalidation

### Database Migrations
Migrations use `PRAGMA user_version` for versioning (see `services/migrations.py`):
- Migrations are idempotent and check for existence before altering
- Auto-backup before migrations (uses `VACUUM INTO`)
- Runs automatically on app startup via `run_migrations()`

To add a new migration:
1. Create `_migration_N_description()` function in `services/migrations.py`
2. Add to `MIGRATIONS` list at bottom of file
3. Migration runs on next app start

### Configuration System
Environment-based config in `config.py`:
- **DevConfig**: DEBUG=True, verbose logging, inline reports, raises on DB errors
- **ProdConfig**: DEBUG=False, background reports, Sentry enabled
- **StagingConfig**: Mixed config for staging environment

Controlled by `APP_ENV` or `FLASK_ENV` environment variable.

### Status Mapping System
Production statuses from OData can be mapped to custom display names:
- Admin route: `/status_mapping` lists all mappings
- `/refresh_statuses`: Syncs from live OData (fetches all unique statuses)
- Inactive statuses are hidden but preserved in DB
- Applied during data processing in `services/buz_data.py`

### Export System (services/export.py)
Downloads support filtering and scrubbing:
- **Filters**: status, group (Customer Group), supplier
- **Scrubbing**: `scrub_sensitive()` removes columns matching cost/margin/buy/wholesale/markup/cogs/pkid patterns
- **Headers**: `ordered_headers()` ensures consistent column order
- **Formats**: XLSX (via pandas/openpyxl) and CSV
- **Formula injection prevention**: `_sanitize_for_excel()` prefixes dangerous values (`=`, `+`, `-`, `@`) with `'`

### Finished Status Filtering (services/buz_data.py)
Orders where ALL lines have a terminal status are excluded from reports. Terminal statuses are defined in `FINISHED_STATUSES` at the top of `buz_data.py`. Update that frozenset when new terminal statuses are added in OData.

## Important Implementation Details

### OData Client (services/odata_client.py)
- Uses `TimeoutSession` with (5, 20) second (connect, read) timeouts
- Retry strategy: 1 retry for 5xx/429, no retries on long reads
- Switches to POST for long filter queries to avoid URL length limits
- Error handling: 5xx treated as hard failures, triggers cache fallback

### Customer vs Customer Group
The `field_type` column in customers table determines data fetching:
- **"Customer Name"**: Calls `get_open_orders(conn, name, instance)`
- **"Customer Group"**: Calls `get_open_orders_by_group(conn, name, instance)`

Both use different OData filters but return the same data structure.

### Error Handling
- Sentry integration with `before_send` hook to scrub sensitive data and deduplicate errors (60s window, max 200 tracked keys)
- Dev mode: raises exceptions for debugging
- Prod mode: returns error templates (400.html, 403.html, 404.html, 500.html, etc.)
- Job errors captured with `update_job(job_id, error=str(exc))`

### URL Validation
All routes accepting `<obfuscated_id>` are validated by a `before_request` hook — only 32-character lowercase hex strings (UUID hex format) are accepted. Invalid IDs return 404.

### Authentication
- Google OAuth via Authlib
- Users must exist in `users` table and have `active=1`
- Roles: "admin" (full access) or "user" (limited access)
- Use `@role_required("admin", "user")` decorator for route protection

### Testing Patterns
Fixtures in `tests/conftest.py`:
- `app`: Flask app instance with TESTING=True
- `client`: Test client for making requests
- `logged_in_admin`: Mocks authenticated admin user
- `_env`: Sets up minimal environment variables and disables Sentry

Common patterns:
```python
def test_route(client, logged_in_admin):
    resp = client.get("/admin")
    assert resp.status_code == 200
```

### CI
GitHub Actions runs `pytest` on push and PR to `main` against Python 3.11 and 3.12. See `.github/workflows/ci.yml`.

## Environment Variables

Required for all environments:
- `DATABASE`: Path to SQLite database file
- `FLASK_SECRET`: Secret key for session management
- `GOOGLE_CLIENT_ID`: Google OAuth client ID
- `GOOGLE_CLIENT_SECRET`: Google OAuth client secret
- `BUZ_DD_USERNAME`, `BUZ_DD_PASSWORD`: DD instance credentials
- `BUZ_CBR_USERNAME`, `BUZ_CBR_PASSWORD`: CBR instance credentials

Production-only:
- `SERVER_NAME`: Server hostname for OAuth redirects
- `SENTRY_DSN`: Sentry error tracking (optional)

Optional:
- `APP_ENV`: Environment (development/production/staging)
- `SENTRY_DISABLED`: Set to "1" to disable Sentry
- `BUZ_API_KEY`: API key for REST API authentication (required for /api/v1/* endpoints except /health)

## Common Gotchas

1. **Database connections**: Always use `get_db()` for request context or pass explicit `conn` parameter for background threads
2. **OData timeouts**: If adding new endpoints, use the `TimeoutSession` pattern from `odata_client.py`
3. **CSV/XLSX exports**: Always call `scrub_sensitive()` before `ordered_headers()` to maintain security
4. **Migrations**: Make them idempotent using `_column_exists()` and `_object_exists()` checks
5. **Job progress**: Never let `progress()` callbacks break the main job; wrap in try/except
6. **Cache keys**: Use descriptive prefixes like `statuses:{instance}` or `orders:{customer}:{instance}`
7. **Timezone**: The app uses `Australia/Sydney` timezone for scheduling (see `prewarm.sh`)
8. **Job log updates**: `update_job()` uses atomic `json_insert` in SQLite — do not read-modify-write the log column
9. **obfuscated_id format**: Must be a 32-char lowercase hex string (UUID hex). Validated by `before_request` hook
10. **User input validation**: `/add_user` validates email contains `@`, name is non-empty, and role is in `{"admin", "user"}`
11. **OData logging**: Filter queries are NOT logged (PII risk). Only the endpoint URL and filter count are logged

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/derekgwatson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
