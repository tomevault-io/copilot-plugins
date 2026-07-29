## unifi-toolkit

> **Analysis Date:** 2026-03-18

# Coding Conventions

**Analysis Date:** 2026-03-18

## Naming Patterns

**Files:**
- Use `snake_case.py` for all Python modules: `unifi_client.py`, `unifi_session.py`, `url_validator.py`
- Test files prefixed with `test_`: `test_auth.py`, `test_cache.py`, `test_crypto.py`
- Routers named by domain: `auth.py`, `config.py`, `events.py`, `devices.py`, `webhooks.py`
- Database models in `database.py` (SQLAlchemy) or `models.py` (Pydantic) per tool

**Functions:**
- Use `snake_case` for all functions: `get_gateway_info()`, `decrypt_password()`, `run_migrations()`
- Private/internal functions prefixed with underscore: `_repair_schema()`, `_normalize_timestamp()`, `_parse_legacy_ips_event()`, `_add_missing_columns()`, `_is_expired()`
- Getters use `get_` prefix: `get_settings()`, `get_database()`, `get_scheduler()`, `get_cipher()`
- Setters use `set_` prefix: `set_gateway_info()`, `set_ips_settings()`
- Boolean checkers use `is_` prefix: `is_auth_enabled()`, `_is_expired()`
- Factory functions use `create_` prefix: `create_app()`, `create_session()`

**Variables:**
- Use `snake_case` for all variables: `gateway_info`, `ips_settings`, `password_hash`
- Constants use `UPPER_SNAKE_CASE`: `CACHE_TTL_SECONDS`, `DEFAULT_REFRESH_INTERVAL`, `RETENTION_DAYS`, `RATE_LIMIT_WINDOW`
- Private module-level state prefixed with underscore: `_scheduler`, `_settings`, `_database`, `_cache`, `_sessions`

**Types/Classes:**
- Use `PascalCase` for classes: `UniFiClient`, `ThreatEvent`, `ToolkitSettings`, `SecurityHeadersMiddleware`
- SQLAlchemy models are singular nouns: `ThreatEvent`, `UniFiConfig`, `ThreatIgnoreRule`
- Pydantic models use descriptive suffixes: `ThreatEventResponse`, `WebhookCreate`, `WebhookUpdate`, `IgnoreRuleResponse`
- Pydantic CRUD pattern: `{Entity}Create`, `{Entity}Update`, `{Entity}Response`, `{Entity}ListResponse`

## Code Style

**Formatting:**
- Black formatter with line length 100 (configured in `pyproject.toml`)
- Target Python versions: 3.9, 3.10, 3.11, 3.12

**Linting:**
- Ruff linter with line length 100, target Python 3.9 (configured in `pyproject.toml`)
- mypy configured with `warn_return_any = true`, `warn_unused_configs = true`, `disallow_untyped_defs = false`

**Line Length:** 100 characters max

**Quotes:** Double quotes for strings throughout

**Trailing Commas:** Used in multi-line function calls, dicts, and lists

## Import Organization

**Order:**
1. Standard library imports (`os`, `sys`, `logging`, `datetime`, `pathlib`, `typing`)
2. Third-party imports (`fastapi`, `sqlalchemy`, `pydantic`, `aiohttp`, `bcrypt`)
3. Local application imports (`shared.config`, `shared.database`, `tools.threat_watch.models`)

**Path Style:**
- Absolute imports throughout: `from shared.config import get_settings`
- No relative imports used
- Individual items imported by name: `from sqlalchemy import select, func, desc`

**Common Import Patterns:**
```python
# Logging setup at module top
import logging
logger = logging.getLogger(__name__)

# Type hints from typing
from typing import Optional, Dict, List, Any, AsyncGenerator

# FastAPI router pattern
from fastapi import APIRouter, Depends, HTTPException, Query
router = APIRouter(prefix="/api/events", tags=["events"])

# Database session dependency
from shared.database import get_db_session
```

## Module Structure

**Every Python module starts with a docstring:**
```python
"""
Brief description of the module's purpose
"""
```

**Standard module layout:**
1. Module docstring
2. Standard library imports
3. Third-party imports
4. Local imports
5. Module-level constants
6. Module-level logger: `logger = logging.getLogger(__name__)`
7. Private state variables (prefixed with `_`)
8. Classes and functions
9. Singleton getters at bottom

## Singleton Pattern

Use module-level private variables with getter functions for singletons:

```python
# In shared/config.py
_settings: Optional[ToolkitSettings] = None

def get_settings() -> ToolkitSettings:
    global _settings
    if _settings is None:
        _settings = ToolkitSettings()
    return _settings
```

This pattern is used consistently for: `get_settings()` in `shared/config.py`, `get_database()` in `shared/database.py`, `get_scheduler()` in scheduler modules, `get_ws_manager()` in `shared/websocket_manager.py`.

## Error Handling

**API Endpoints:**
- Use `HTTPException` for client errors (400, 404):
  ```python
  raise HTTPException(status_code=404, detail="Event not found")
  raise HTTPException(status_code=400, detail="Either password or api_key must be provided")
  ```
- Catch generic exceptions, log with `exc_info=True`, return error detail:
  ```python
  except Exception as e:
      logger.error(f"Failed to save UniFi config: {type(e).__name__}: {e}", exc_info=True)
      raise HTTPException(status_code=500, detail=f"Failed to save configuration: {type(e).__name__}: {str(e)}")
  ```
- Re-raise `HTTPException` before catching generic `Exception`:
  ```python
  except HTTPException:
      raise
  except Exception as e:
      ...
  ```

**Background Tasks / Schedulers:**
- Log errors and continue (don't crash the scheduler):
  ```python
  except Exception as e:
      logger.error(f"Error delivering webhook: {e}", exc_info=True)
      return False
  ```

**Startup Errors:**
- Print to stdout with banner formatting and `sys.exit(1)`:
  ```python
  print("=" * 70)
  print("ERROR: ENCRYPTION_KEY not set in .env file!")
  print("=" * 70)
  sys.exit(1)
  ```

**Connection Errors:**
- Return structured error responses (not exceptions) for UniFi connection issues:
  ```python
  return {"configured": True, "connected": False, "error": "Failed to connect"}
  ```

## Logging

**Framework:** Python `logging` module via `logging.getLogger(__name__)`

**Patterns:**
- Use `logger.info()` for significant state changes (startup, config saved, scheduler started)
- Use `logger.debug()` for cache hits, intermediate steps, diagnostic info
- Use `logger.warning()` for recoverable issues (update check failed, migration sync)
- Use `logger.error()` for failures, with `exc_info=True` for stack traces
- Use f-strings in log messages: `logger.info(f"Cached gateway info: {data.get('gateway_name')}")`

**Log Format:** `%(asctime)s - %(name)s - %(levelname)s - %(message)s`

## Comments

**When to Comment:**
- Explain WHY, not WHAT: `# is_unifi_os is auto-detected during connection, default to False for storage`
- Note deprecated fields: `# Deprecated: is_unifi_os is now auto-detected during connection`
- Document API quirks: `# inner_alert_signature`, `# inner_alert_category / catname`
- Mark known issues or workarounds

**Docstrings:**
- Use triple-double-quote docstrings on all public functions and classes
- Google-style format with Args/Returns/Yields sections:
  ```python
  def encrypt_password(password: str) -> bytes:
      """
      Encrypt a password using Fernet symmetric encryption

      Args:
          password: Plain text password to encrypt

      Returns:
          Encrypted password as bytes
      """
  ```
- Short one-line docstrings for simple functions: `"""Get the global settings instance (singleton pattern)"""`

## Function Design

**Size:** Most functions are 10-40 lines. Endpoint handlers can be longer (50-100 lines) when building complex queries.

**Parameters:**
- Use type hints on all function parameters and return types
- Use `Optional[str]` for nullable parameters, not `str | None` (Python 3.9 compat)
- Use Pydantic `Field()` for request model validation with `description`, `ge`, `le`
- Use FastAPI `Query()` for endpoint query parameters with descriptions and validation

**Return Values:**
- Return Pydantic model instances from API endpoints
- Return `dict` from internal data-fetching functions
- Return `bool` from webhook delivery functions
- Return `Optional[T]` from cache getters (None when not cached or expired)

## Pydantic Models

**Request/Response Pattern:**
- `{Entity}Create` for POST request bodies
- `{Entity}Update` for PATCH/PUT with all fields `Optional`
- `{Entity}Response` for single-item responses with `class Config: from_attributes = True`
- `{Entity}ListResponse` for paginated lists with `total`, `page`, `page_size`, `has_more`

**DateTime Serialization:**
- Use custom `serialize_datetime()` helper for ISO format with `Z` suffix
- Apply via `@field_serializer('timestamp')` on datetime fields

**Generic Responses:**
- `SuccessResponse(success: bool, message: Optional[str])`
- `ErrorResponse(error: str, details: Optional[str])`

## Database Patterns

**SQLAlchemy Models:**
- Inherit from `shared.models.base.Base`
- Define `__tablename__` explicitly
- Use `Column()` with explicit `nullable`, `default`, `index` params
- Add composite indexes via `__table_args__`
- Include `__repr__` method

**Session Usage (async):**
- Use FastAPI `Depends(get_db_session)` for endpoint injection
- Use `get_database().get_session()` in scheduler/background tasks
- Always use `select()` with `await db.execute()` — never use `db.query()`

**Common Query Pattern:**
```python
result = await db.execute(select(Model).where(Model.id == value))
item = result.scalar_one_or_none()
```

## Router Patterns

**Router Definition:**
```python
router = APIRouter(prefix="/api/events", tags=["events"])
```

**Endpoint Signatures:**
- Use `response_model` for typed responses
- Use `Depends(get_db_session)` for database access
- Use `Query()` with descriptions for query parameters
- Use Pydantic models for request bodies

## Version Management

Keep version in sync across three files:
- `pyproject.toml` -> `version = "X.Y.Z"`
- `app/__init__.py` -> `__version__ = "X.Y.Z"`
- `app/main.py` -> `version="X.Y.Z"` (FastAPI constructor)

---

*Convention analysis: 2026-03-18*

---
> Source: [Crosstalk-Solutions/unifi-toolkit](https://github.com/Crosstalk-Solutions/unifi-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
