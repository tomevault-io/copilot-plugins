## backchannel

> Python API conventions for the FastAPI backend


# API Conventions

## FastAPI Patterns

- All routes defined in `api/server.py` under the `/api` prefix
- Use Pydantic `BaseModel` for request bodies
- Raise `HTTPException` for error responses
- CORS allows `localhost:5173` and `localhost:3000`

## Database

- Use `get_db()` from `api.db` — returns a thread-local `sqlite3.Connection`
- Always use parameterized queries (`?` placeholders), never f-strings
- Rows are `sqlite3.Row` (dict-like access via `row["column"]`)
- Migrations live in `init_db()` as idempotent ALTER/CREATE IF NOT EXISTS
- WAL journal mode and foreign keys are enabled at connection time

## Service Pullers

New integrations follow this pattern:

1. Subclass `BasePuller` from `api.pullers.base`
2. Implement `test_connection()`, `pull(cursor, since)`, and `normalize(raw_item)`
3. Return a `PullResult` dataclass from `pull()`
4. Register via `register_puller("service_type", MyPuller)` in `api/services/manager.py`

## Service Manager

- `api/services/manager.py` handles connect, disconnect, sync, CRUD
- Pullers are looked up from `PULLER_REGISTRY` by `service_type`
- Credentials are JSON-serialized in the `services` table

## Style

- Use `logging.getLogger(__name__)` for module-level loggers
- Type hints on function signatures (`str | None` union syntax, not `Optional`)
- Imports from the `api` package use absolute paths (e.g., `from api.db import get_db`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shaunsales)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shaunsales)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
