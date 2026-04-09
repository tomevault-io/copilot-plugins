## ragflow-scraper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Local development (preferred — Docker is not always running)

```bash
# Lint
ruff check app/ tests/

# Run all unit tests (excludes Docker-only test files)
BASIC_AUTH_ENABLED=true .venv/bin/python -m pytest tests/unit/ \
  --ignore=tests/unit/api_routes/test_basic_auth.py \
  --ignore=tests/unit/scrapers/test_scraper_registry.py -v

# Run a single test file
BASIC_AUTH_ENABLED=true .venv/bin/python -m pytest tests/unit/clients/test_paperless_client.py -v

# Run a single test
BASIC_AUTH_ENABLED=true .venv/bin/python -m pytest tests/unit/clients/test_paperless_client.py::TestOwnerPayload::test_returns_owner_dict -v

# Type checking
.venv/bin/pyright app/

# Stack tests (live services on Unraid 192.168.1.101)
DOTENV_PATH=.env.stack NODE_ENV=test .venv/bin/python -m pytest tests/stack -v
```

### CI parity (what GitHub Actions runs)

```bash
# Lint
ruff check app/ tests/

# Unit tests (CI env vars)
FLASK_ENV=testing BASIC_AUTH_ENABLED=false LOG_TO_FILE=false \
  SECRET_KEY=ci-test-secret-key-not-for-production \
  .venv/bin/python -m pytest tests/unit/ -m "not slow" -v --cov=app --cov-report=xml

# Integration tests
FLASK_ENV=testing BASIC_AUTH_ENABLED=false LOG_TO_FILE=false \
  SECRET_KEY=ci-test-secret-key-not-for-production \
  .venv/bin/python -m pytest tests/integration/ -v --tb=short

# Security audit
pip-audit
```

### Docker (when available)

```bash
make dev-up          # Start dev stack
make test-unit       # Run unit tests in container
make test-file FILE=tests/unit/clients/test_paperless_client.py  # Single file
make lint
```

## Architecture

### Pipeline flow

Scraper → `clean_article_html()` → `build_article_html()` → `inline_images()` → disk
→ Pipeline reads HTML → `inject_metadata_stamp()` → Gotenberg (HTML→PDF) → Paperless-ngx archive
→ Parser (Docling/Tika) extracts markdown + metadata → RAG backend (RAGFlow/AnythingLLM/pgvector)

### Backend abstraction (`app/backends/`)

Four backend types, each with an ABC and swappable implementations selected by env var:

| Type | Env var | Options |
|------|---------|---------|
| Parser | `PARSER_BACKEND` | `docling`, `docling_serve`, `tika`, `mineru` |
| Archive | `ARCHIVE_BACKEND` | `paperless`, `s3`, `local` |
| RAG | `RAG_BACKEND` | `ragflow`, `anythingllm`, `pgvector` |
| Vector Store | `VECTOR_BACKEND` | `pgvector` |

Backends are instantiated via `BackendRegistry` (`app/services/backend_registry.py`) which uses `(type, name) → factory` lookup. Factories receive the `ServiceContainer` instance and call `container._get_effective_url()` / `_get_config_attr()` — never import `Config` directly. This preserves test patches on `app.services.container.Config`.

### Dependency injection

`ServiceContainer` (`app/services/container.py`) is a singleton with lazy-loaded properties. Access via `from app.container import get_container`. All external service clients are properties on the container.

### Scraper system (`app/scrapers/`)

- `BaseScraper`: abstract base with mixin composition (IncrementalState, ExclusionAndMetadata, CloudflareBypass, WebDriverLifecycle)
- `FlareSolverrPageFetchMixin`: Cloudflare bypass (replaces Selenium)
- `ScraperRegistry`: auto-discovers scrapers via subclass detection
- Scrapers set `skip_webdriver = True` and add `FlareSolverrPageFetchMixin` to MRO

### Web UI (`app/web/`)

Flask + HTMX, blueprint-based. Settings split into sub-package: `settings_ui`, `settings_api`, `settings_recon`. Security: CSRF (Flask-WTF), Basic Auth (optional), rate limiting, security headers.

### Configuration

- `app/config.py`: static config from env vars, validated at startup
- `config/settings.json`: runtime settings tunable via web UI
- `SettingsManager`: thread-safe read/write with `threading.RLock`

## Testing patterns

### Critical test requirements

- Use `.venv/bin/python`, not `python` (no system python on this host)
- `pytest-timeout` is NOT installed — don't use `--timeout`
- `test_basic_auth.py` and `test_scraper_registry.py` fail locally due to missing `/app/data/logs/` — exclude with `--ignore`

### Mock pitfalls

- **Always set `mock_response.headers`**: `Mock().headers.get()` returns a truthy Mock, breaking content-type branching. Fix: `mock_response.headers = {"Content-Type": "text/plain"}`
- **Config patching**: use `patch.object(Config, "ATTR", value)` — module-level `from X import Config` bindings aren't updated by `patch("module.Config")`
- **`or` fallback chains**: `api_url or Config.X or ""` causes empty-string args to fall through to real `.env` values. Patch Config in the module under test instead.
- **Fixture `yield` not `return`**: inside `with patch(...)`, `return` exits the context manager before the test runs. Always `yield`.
- **`DEFAULT_SETTINGS.copy()` is shallow**: nested dicts share references. Use `copy.deepcopy()`.
- **BaseScraper local import**: `__init__` does `from app.container import get_container`, so patch target is `app.container.get_container`
- **Module-level `get_logger()`**: causes `/app/data/logs/` FileNotFoundError during collection. Logger must be in `__init__`, not module level.

### Flask integration test fixtures

Tests calling `create_app()` must:
1. Patch `container` and `job_queue` BEFORE app creation
2. Set `WTF_CSRF_ENABLED = False` (CSRF state leaks between tests)
3. Patch `Config.SECRET_KEY` (templates use `csrf_token()` which needs sessions)
4. Use `yield` (not `return`) to keep patches active

### Settings blueprint tests

Must patch `container` in ALL 4 submodules: `ui`, `api`, `helpers`, `reconciliation`.

## Paperless-ngx API

- `/api/tasks/{id}/` does NOT exist — use `/api/tasks/` (list) and filter by `task_id`
- Task lifecycle: PENDING → STARTED → SUCCESS (with `related_document`)
- Custom fields require separate `PATCH /api/documents/{id}/` after upload
- `CUSTOM_FIELD_MAPPING` in `paperless_client.py` maps metadata keys → (field name, data type)

## Ruff config (`ruff.toml`)

Line length 150, Python 3.11, rules: E + F + S. Key ignores: S101 (assert for pyright), S104 (0.0.0.0 bind), S110 (try-except-pass cleanup), S311 (random jitter). Tests allow hardcoded passwords and assert.

## Pyright

`pyrightconfig.json` has `"venvPath": "."` and `"venv": ".venv"`. Use `# type: ignore[assignment]` for `getattr()` + `callable()` dynamic dispatch, `# type: ignore[import-not-found]` for optional deps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nkissick-del)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nkissick-del)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
