## keystrokes-dynamic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Flask web app for **behavioral biometric authentication using keystroke dynamics**. The user types their password and the system verifies both the password *and* the typing rhythm against per-user ML models.

## Common commands

Run locally (Windows PowerShell, venv already exists):

```powershell
.\venv\Scripts\python.exe run.py
```

Or via Docker (binds `./data` → `/app/data` for SQLite persistence, port 5000):

```powershell
docker compose up --build -d
docker compose logs -f app
```

Smoke test (this branch has no `tests/` dir — `quick_smoke.py` is the canonical end-to-end check):

```powershell
.\venv\Scripts\python.exe scripts\quick_smoke.py
# Expect final line: SMOKE_CHECK: PASS
```

Database migrations (Flask-Migrate / Alembic, dir = `migrations/`):

```powershell
$env:FLASK_APP="run.py"
flask db migrate -m "message"
flask db upgrade
```

> The factory detects `flask db ...` invocations and **skips `db.create_all()`** so migrations are not corrupted by implicit table creation. See [_is_running_db_cli](app/__init__.py#L103-L120) and [_should_run_create_all](app/__init__.py#L123-L129).

Frontend CSS (Tailwind) — only needed when editing styles:

```powershell
npm run build:css         # one-shot
npm run build:css:watch   # dev watch
```

Production entrypoint (Railway/Heroku): `gunicorn run:app` — see [Procfile](Procfile).

## Architecture

### Application factory + blueprint layout

[run.py](run.py) calls `create_app(FLASK_ENV)` in [app/__init__.py](app/__init__.py). The factory:

1. Loads a config class from [config.py](config.py) (`DevelopmentConfig` / `ProductionConfig` / `TestingConfig`) via `get_config()`, then runs `validate_config()` (threshold bounds, sample counts, SameSite, prod SECRET_KEY).
2. Applies `ProxyFix` (required behind Railway/Heroku reverse proxy — without it Talisman causes an HTTPS redirect loop).
3. Initializes extensions: SQLAlchemy, Flask-Migrate, Flask-Login, CSRFProtect, Flask-Limiter (Redis-backed if `REDIS_URL` set), Flask-Caching, Flask-SocketIO, Flask-Mail.
4. Registers six blueprints in this order: `main_bp`, `auth_bp`, `api_bp` (`/api`), `admin_bp` (`/admin`), `dataset_bp`, `health_bp` (`/health`).
5. **CSRF is exempted only on the `api` blueprint.** Admin AJAX calls rely on `base.html`'s global `fetch()` override to auto-inject `X-CSRFToken`.
6. In `production`, Talisman is enabled with a strict CSP that uses **per-request nonces** — inline `<script>` tags must carry `nonce="{{ csp_nonce() }}"`. The factory injects a no-op `csp_nonce` into templates in dev so the same templates render in both modes ([inject_dev_flags](app/__init__.py#L244-L255)).

### API blueprint is a package, not a file

[app/blueprints/api/](app/blueprints/api/) splits routes across `enrollment.py`, `login_core.py`, `login_verify.py`, `two_factor.py`, `user.py`, `verification.py`, `dataset.py`, `partner.py`. All decorate the **same `api_bp` object** imported from [_shared.py](app/blueprints/api/_shared.py). When adding an API route, put it in the topic file that fits and ensure that module is imported by [api/__init__.py](app/blueprints/api/__init__.py) — otherwise the decorators never run.

### Service resolution (ServiceRegistry + proxy)

[_shared.py](app/blueprints/api/_shared.py) does **not** instantiate services directly. It registers lazy providers on a `ServiceRegistry` and exposes `auth_service` / `biometric_service` as `_ServiceProxy` objects that resolve at attribute-access time via [resolve_service](app/services/resolution.py). The registry is attached to `app.extensions["service_registry"]` ([_attach_service_registry](app/__init__.py#L96-L100)) so tests can override services per-app without rebinding module-level singletons. **Do not bypass the proxy** by importing concrete classes into route modules; that breaks override semantics.

### ML backend is mode-switched, not pluggable per request

Internal login flow honors `ML_BACKEND` env / config: `rf` (default, RandomForest), `svm`, or `statistical` (template-distance, no training; aliases `stat`/`template` normalize to it). The factory normalizes the value at startup ([app/__init__.py:172-174](app/__init__.py#L172-L174)) and `BiometricService._normalize_backend_name` enforces the same set at runtime.

**Partner API (`/api/partner/...`) always uses statistical/template-distance comparison and ignores `ML_BACKEND`.** Partner pass/fail uses `PARTNER_DECISION_THRESHOLD` (default 0.7), separate from the internal `VERIFICATION_THRESHOLD`.

Per-user trained models live in the `UserMLModel` table; raw samples in `UsersVector` (legacy aliases `KeystrokeVector`/`FeatureVector`/`EnrollmentVector` still re-exported — prefer `UsersVector` in new code, see [app/models/__init__.py](app/models/__init__.py#L17-L19)).

### Identifiers and DB

Primary keys are **UUIDv7** (`uuid6` package). On SQLite, `quick_smoke.py` registers a compiler hook to map `UUID` → `CHAR(32)` ([scripts/quick_smoke.py:17-19](scripts/quick_smoke.py#L17-L19)) — keep this in mind when writing other smoke/test entrypoints.

Default DB is **SQLite at `data/biometric_auth.db`**. Set `DATABASE_TYPE=postgresql` + `DATABASE_URL=...` for Postgres.

### Public dataset-only lockdown

Setting `DATASET_ONLY=1` flips a `before_request` guard that returns 404 for everything except `/dataset`, `/api/dataset/`, `/static/`, `/health/` ([_configure_dataset_only_guard](app/__init__.py#L132-L145)). Used for public data-collection deployments where login/admin must be hidden.

### Dev rate-limit escape hatch

`DEV_LENIENT_RATELIMIT=True` disables server-side Flask-Limiter and triggers lenient client-side retry behavior in templates (via `inject_dev_flags`). Use this when iterating on the keystroke capture UI — 429s otherwise make the JS look broken.

## Files to know

- [config.py](config.py) — all env-driven knobs and validation
- [app/__init__.py](app/__init__.py) — factory wiring
- [app/services/biometric_service.py](app/services/biometric_service.py) — backend-routing layer over `ml_model_service` / `svm_model_service` / statistical scorer
- [app/services/registry.py](app/services/registry.py) + [resolution.py](app/services/resolution.py) — service DI plumbing
- [scripts/quick_smoke.py](scripts/quick_smoke.py) — end-to-end auth/dashboard smoke; mirror its patterns when adding new smoke checks
- [DEVELOPER_FIRST_RUN_CHECKLIST.md](DEVELOPER_FIRST_RUN_CHECKLIST.md) — Docker-first onboarding flow
- [ENDPOINT_ANALYSIS.md](ENDPOINT_ANALYSIS.md) — route inventory reference

## This branch is trimmed

Per [README.md](README.md), this branch (`update_apis`) excludes backups, raw datasets, research notebooks, and one-off maintenance scripts. The full snapshot lives on `archive/full-repo-20260420`. Several scripts in `scripts/` (e.g. `_fix_paper.py`, `run_ml_local.py`, `query_db_stats.py`) and the `merge_railway_to_pg.py` / `inspect_railway_db.py` files at root are **gitignored** — they exist locally but should not be relied on for shared workflows.

---
> Source: [Chaizaa/Keystrokes-Dynamic](https://github.com/Chaizaa/Keystrokes-Dynamic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
