## tgma-platform

> TGMA (Tripura Gut Microbiome in Adolescents) — Flask 3.1 research data management platform.

# CLAUDE.md — Project Intelligence for TGMA Platform

## Project Overview
TGMA (Tripura Gut Microbiome in Adolescents) — Flask 3.1 research data management platform.
ICMR-funded study at Tripura University. Self-hosted on Dell PowerEdge R730, LAN-only, Docker deployment.

**Status (April 2026)**: Platform is **feature-complete**. 65+ source files, 44 tests passing, Docker deployment ready. Features: KoboToolbox sync, participant edit/delete UI, demo-data wipe script, label-kit batched reprint (Seznik thermal printer), per-participant Document Vault (consent/assent/info-sheet/questionnaire/photos, PDF+JPG+PNG, role-gated upload/delete).

## Deployment
- **Server**: PowerEdge R730, Ubuntu, Docker Compose at `/home/mmilab/Desktop/tgma-platform`
- **Access (LAN)**: http://192.168.1.35:8100
- **Access (public)**: ngrok HTTPS tunnel — URL changes on restart; get current URL with `sudo docker compose logs ngrok | grep url=` or run `bash scripts/ngrok_url.sh` from the server
- **Database**: PostgreSQL 16 in Docker, volume `tgma-platform_pgdata`
- **Rebuild flow**: `git pull && sudo docker compose down && sudo docker compose up -d --build`
- **Re-seed / upsert users**: `sudo docker compose exec web python scripts/init_db.py`
- **Re-seed with synthetic data**: `sudo docker compose exec web python scripts/init_db.py --synthetic`
- **Wipe demo data (keeps users + schema)**: `sudo docker compose exec web python scripts/wipe_data.py --confirm` — deletes all participants (cascades), ID allocations, and KoboSync logs
- **Full reset (wipes data)**: add `-v` flag to `docker compose down`
- **Dev machine**: Windows (Anaconda), run with `conda run -n base python`
- **Ngrok setup (one-time)**: Sign up at ngrok.com, add `NGROK_AUTHTOKEN=<token>` to `.env` on server, then rebuild

## Build Verification
```bash
# App factory smoke test
conda run -n base python -c "from app import create_app; app = create_app('testing'); print('OK')"

# Full test suite (44 tests)
conda run -n base python -m pytest tests/ -v
```

## Complete File Inventory

### App Core
| File | Purpose |
|------|---------|
| `wsgi.py` | WSGI entry point |
| `config.py` | DevelopmentConfig / ProductionConfig / TestingConfig; study params (TARGET_ENROLLMENT=440, TARGET_SAMPLES_YEAR1=160, SEQUENCING_BATCH_SIZE=32); KOBO_API_URL config |
| `requirements.txt` | All Python deps (Flask 3.1, SQLAlchemy, pandas, python-barcode, gunicorn, etc.) |
| `.env.example` | Template for environment variables (DB, Kobo, upload paths) |
| `app/__init__.py` | App factory — registers all 10 blueprints (incl. kobo, documents), extensions, context processor |
| `app/extensions.py` | SQLAlchemy, Migrate, LoginManager, CSRFProtect |
| `app/auth.py` | Login/logout blueprint, Flask-Login user_loader |

### Models (8 files)
| File | Tables |
|------|--------|
| `app/models/__init__.py` | Re-exports all models |
| `app/models/user.py` | `users` — bcrypt password hashing, roles (pi, co_pi, bioinformatician, field_supervisor) |
| `app/models/participant.py` | `participants` — PK is `tracking_id` (VARCHAR 20), enrollment status, GPS coords |
| `app/models/clinical.py` | `health_screenings`, `anthropometrics`, `menstrual_data` — computed BMI, waist-hip ratio |
| `app/models/survey.py` | `lifestyle_data`, `environment_ses` — diet, activity, SES data from KoboToolbox |
| `app/models/sample.py` | `samples`, `sample_shipments` — freezer positions (UNIQUE constraint), chain of custody |
| `app/models/results.py` | `hormone_results`, `sequencing_results`, `id_allocations` — HOMA-IR, TG/HDL computed props |
| `app/models/admin.py` | `audit_log`, `blood_reports`, `kobo_sync_log`, `participant_documents` — PDF upload storage for diagnostics + sync run history + per-participant scanned forms/photos |

### Route Blueprints (10 files)
| File | URL Prefix | Key Features |
|------|-----------|--------------|
| `app/routes/__init__.py` | — | Package init |
| `app/routes/dashboard.py` | `/` | Stats cards, enrollment progress, district breakdown charts |
| `app/routes/participants.py` | `/participants` | CRUD, server-side DataTables API (`api_data`), detail, edit (POST), delete (POST) — edit/delete gated to PI/Co-PI/Bioinformatician; delete cascades to all child records |
| `app/routes/samples.py` | `/samples` | Register, tracker, freezer map, dispatch, detail; ALLOWED_SAMPLE_TYPES = ['stool', 'saliva_cortisol'] |
| `app/routes/diagnostics.py` | `/diagnostics` | Blood report PDF upload (NOT Excel import) |
| `app/routes/ids.py` | `/ids` | Bulk ID allocation per district; batches table with re-print, Excel kit, per-participant thermal print (50x30mm page / 25mm sticker) and delete buttons; `thermal_labels()` generates 10 QR code PNGs in-memory via `qrcode` + base64 data URIs; `delete_allocation()` and `delete_batch()` with participant-exists safety check; LABEL_KIT = 10 labels (blood vials removed) |
| `app/routes/quality.py` | `/quality` | GPS bounds check, outlier detection (|Z|>3), duplicates, missing data, incomplete sample sets |
| `app/routes/kobo.py` | `/kobo` | Manual "Sync Now" button, full re-sync, sync log history, per-run detail view. Restricted to PI/Co-PI/Bioinformatician. |
| `app/routes/documents.py` | `/documents` | Per-participant document vault — upload/view/download/delete scanned consent/assent/info-sheet/questionnaire forms and photos. Merges read-only `BloodReport` rows into the same view. Upload + delete gated to PI/Co-PI/Bioinformatician. Path-traversal guard via `validate_tracking_id()`. Self-healing `_ensure_table()` for first deploy. |
| `app/routes/ml.py` | `/ml` | Placeholder for ML pipeline status |
| `app/routes/reports.py` | `/reports` | ICMR progress report, export-ready |

### Templates (22 files)
| File | Notes |
|------|-------|
| `app/templates/base.html` | Sidebar layout, Satoshi font, Bootstrap 5, DataTables, Chart.js — all local assets. Sidebar has "Data" section with KoboToolbox Sync link (role-gated). |
| `app/templates/login.html` | Centered login card |
| `app/templates/dashboard.html` | Stat cards, enrollment chart, district pie |
| `app/templates/participants/list.html` | Server-side DataTables with district/gender/status/lifestyle filters |
| `app/templates/participants/detail.html` | Tabbed view: demographics, clinical, samples, surveys. Header has role-gated Edit (Bootstrap collapse form) and Delete (confirmation modal) buttons + Documents vault shortcut. |
| `app/templates/samples/tracker.html` | Sample pipeline overview |
| `app/templates/samples/register.html` | New sample form |
| `app/templates/samples/detail.html` | Single sample view |
| `app/templates/samples/freezer.html` | Freezer grid map |
| `app/templates/samples/dispatch.html` | Shipment management |
| `app/templates/diagnostics/index.html` | Blood report PDF upload |
| `app/templates/ids/allocate.html` | Bulk ID generation; batches table with expandable per-participant thermal print + delete buttons (per-ID and per-batch) |
| `app/templates/ids/labels.html` | Batch label sheet — 10-label info banner, per-participant thermal print button table, 4-column grid preview |
| `app/templates/ids/thermal_labels.html` | Print-ready thermal labels — `@page { size: 50mm 30mm }` with 25mm inner content area (actual sticker size), 10 QR codes as base64 PNGs, QR left + text right layout, 3x3 screen preview |
| `app/templates/quality/dashboard.html` | Data quality dashboard (GPS, outliers, duplicates, completeness) |
| `app/templates/ml/status.html` | Coming soon placeholder |
| `app/templates/reports/index.html` | Report listing |
| `app/templates/reports/icmr_progress.html` | ICMR progress report template |
| `app/templates/kobo/sync.html` | KoboToolbox sync dashboard — Sync Now / Full Re-sync buttons, sync history table, latest run details preview |
| `app/templates/kobo/log_detail.html` | Per-run detail view — stat cards, filterable submission table (new/updated/skipped/error), error messages |
| `app/templates/documents/index.html` | Document vault landing — participant list with blood-report + other-doc counts, client-side search |
| `app/templates/documents/vault.html` | Per-participant vault — role-gated upload form, grouped cards per doc_type, merged blood-report section |

### Static Assets
| File | Notes |
|------|-------|
| `app/static/css/custom.css` | Full custom theme — Satoshi font, green palette (#2D6A4F), sidebar, cards, tables, responsive, print |
| `app/static/js/charts.js` | Chart.js helpers for dashboard |
| `app/static/vendor/bootstrap.min.css` | Bootstrap 5 (local) |
| `app/static/vendor/bootstrap.bundle.min.js` | Bootstrap 5 JS (local) |
| `app/static/vendor/jquery.min.js` | jQuery 3 (local) |
| `app/static/vendor/chart.min.js` | Chart.js (local) |
| `app/static/vendor/datatables.min.js` | DataTables (local) |
| `app/static/vendor/datatables.min.css` | DataTables CSS (local) |
| `app/static/vendor/fonts/Satoshi-*.woff2` | Satoshi font family (Light, Regular, Medium, Bold) |

### Utils
| File | Purpose |
|------|---------|
| `app/utils/__init__.py` | Package init |
| `app/utils/helpers.py` | `validate_tracking_id()`, `generate_tracking_id()`, `generate_sample_id()`, `validate_gps()`, `validate_age()`; TRACKING_ID_PATTERN = `TGMA-(WT|ST|DL)-(M|F)-(\d{3,4})` (3-digit new, 4-digit legacy); SAMPLE_SUFFIXES dict (STL, BLD, SLV1-4, COR, DNA, SRM) |
| `app/utils/decorators.py` | `@role_required()` decorator for route protection |
| `app/utils/audit.py` | Audit logging helper |

### ETL Scripts (3 files)
| File | Purpose |
|------|---------|
| `etl/kobo_sync.py` | KoboToolbox sync engine — `_do_sync()` core logic + `run_sync()` context-aware wrapper (uses `flask.has_app_context()` to decide whether to push new context or reuse existing). Paginated API fetch, critical-field validation, idempotent upsert, sync log with per-submission details. Supports `--full` re-sync. |
| `etl/hormone_import.py` | Import hormone/diagnostics results from Excel/CSV — validates ranges, maps columns, supports `--dry-run` |
| `etl/sequencing_import.py` | Import Nucleome Informatics vendor manifest TSV — per-sample sequencing stats into `sequencing_results` |

### Deployment (5 files)
| File | Purpose |
|------|---------|
| `Dockerfile` | python:3.12-slim, libpq-dev + gcc only, WeasyPrint excluded via grep, gunicorn (4 workers), non-root user not yet added |
| `docker-compose.yml` | PostgreSQL 16-alpine + Flask/Gunicorn; DB port NOT exposed (mistake #9); `WEB_PORT` configurable (default 8100); static volume mount |
| `nginx.conf` | Reverse proxy config for optional Nginx in front of Gunicorn; LAN-only bind |
| `systemd/tgma-dashboard.service` | Systemd unit file for non-Docker deployment option |
| `scripts/backup.sh` | Daily pg_dump with 30-day retention |

### Scripts
| File | Purpose |
|------|---------|
| `scripts/init_db.py` | DB init + **upsert** 5 users (update existing, create new) + `--synthetic` flag for ~50 test participants. **Contains real user credentials — repo must be PRIVATE.** |
| `scripts/wipe_data.py` | One-shot reset — deletes all participants (cascades), ID allocations, and KoboSync logs. Requires `--confirm` flag. Preserves users and schema. Also removes `{UPLOAD_FOLDER}/participants/` subtree from disk; `blood_reports/` is preserved. |
| `scripts/init_db.sql` | PostgreSQL extensions (pg_trgm). Mounted in Docker entrypoint. |
| `scripts/generate_barcodes.py` | Generate Code128 barcode label PDFs using python-barcode. Supports `--ids`, `--range`, `--samples`, `--from-db`. |
| `scripts/ngrok_url.sh` | Fetch current ngrok public URL from local ngrok API (port 4040). Run on server after deploy. |
| `scripts/parse_kobo.py` | Utility to parse KoboToolbox form JSON and list survey fields (not committed) |
| `scripts/test_kobo_route.py` | Dev-only smoke test for `/kobo/` route — creates in-memory DB, logs in as PI, checks 200 response. Not committed to repo. |

### Tests (4 files, 44 tests total)
| File | Tests |
|------|-------|
| `tests/conftest.py` | App factory with TestingConfig (SQLite in-memory), session rollback per test, `pi_user` and `auth_client` fixtures |
| `tests/test_models.py` | 19 tests — tracking ID validation, sample ID generation, GPS bounds, age validation, BMI/WHR computation, HOMA-IR, TG/HDL ratio, password hashing, role checks |
| `tests/test_auth.py` | 6 tests — login page, success/failure login, protected route redirect, logout |
| `tests/test_kobo_sync.py` | 19 tests — critical-field validation (missing tracking_id/name/gender/district rejected), field mapping, optional NULL sections accepted, gender/district derived from tracking_id, age range enforcement, GPS geolocation parsing, menstrual data gender-gating, safe type conversion helpers |

## Mistakes Log — Do NOT Repeat

### 1. SESSION_COOKIE_SECURE on HTTP
**What happened**: Set `SESSION_COOKIE_SECURE = True` in ProductionConfig. This caused CSRF "session token missing" errors because the cookie was only sent over HTTPS, but the server runs plain HTTP on LAN.
**Fix**: Set `SESSION_COOKIE_SECURE = False` for LAN-only HTTP deployment.
**Rule**: Never enable secure cookies unless HTTPS is configured.

### 2. Special characters in DATABASE_URL password
**What happened**: DB_PASSWORD in `.env` contained `&`, `%`, `@` characters. Docker Compose interpolated these into `DATABASE_URL`, breaking URL parsing. psycopg2 saw `E&%u@db` as hostname.
**Fix**: Use only alphanumeric passwords in `.env` (no `@`, `&`, `%`, `#`, `=`).
**Rule**: Always use simple alphanumeric passwords for database URLs in environment variables.

### 3. Dockerfile apt-get failures on slim images
**What happened**: `apt-get update` failed with exit code 100 inside `python:3.12-slim` when installing WeasyPrint dependencies (libpango, libpangocairo, etc.).
**Fix**: Removed WeasyPrint deps (not needed yet), kept only `libpq-dev` and `gcc` for psycopg2.
**Rule**: Keep Dockerfile dependencies minimal. Only add system packages that are actively needed.

### 4. Inline grep in Dockerfile RUN
**What happened**: `RUN pip install --no-cache-dir $(grep -v WeasyPrint requirements.txt)` failed because shell expansion inside Docker RUN doesn't work as expected with multiline output.
**Fix**: Split into two commands: `grep > reqs.txt && pip install -r reqs.txt`.
**Rule**: Never use command substitution `$(...)` with pip install in Dockerfiles. Write to temp file first.

### 5. PostgreSQL init.sql referencing non-existent tables
**What happened**: `init_db.sql` creates a trigram index on `participants` table, but this runs during DB initialization before Flask creates the tables. Caused the DB container to be marked "unhealthy".
**Fix**: The error is harmless (DB still starts), but the healthcheck timed out on first run. Just restart.
**Rule**: Don't put table-dependent SQL in `docker-entrypoint-initdb.d/`. Run indexes after `db.create_all()`.

### 6. Duplicate freezer positions in synthetic data
**What happened**: Random freezer position generation (rack/shelf/box/row/col) created collisions, violating the UNIQUE constraint `uq_samples_freezer_position`.
**Fix**: Track used positions in a `set()` and retry until unique.
**Rule**: Always track uniqueness when generating random data for columns with UNIQUE constraints.

### 7. conda multiline argument on Windows
**What happened**: `conda run -n base python -c "..."` with multiline Python code fails on Windows with "Support for scripts where arguments contain newlines not implemented."
**Fix**: Write Python code to a `.py` file and run it, or use single-line `-c` commands.
**Rule**: Never use multiline strings with `conda run` on Windows.

### 8. GitHub password auth rejected
**What happened**: Tried `git clone` with username/password on the server. GitHub no longer accepts passwords.
**Fix**: Made repo public temporarily for cloning, or use Personal Access Token.
**Rule**: Use PAT tokens or SSH keys for GitHub auth. Never attempt password auth.

### 9. Docker port conflicts on shared server
**What happened**: Original `docker-compose.yml` bound PostgreSQL to port 5432 and Nginx to 80/443 — all already in use by other services.
**Fix**: Removed bundled nginx, removed DB port binding (internal only), exposed web on configurable `WEB_PORT` (8100).
**Rule**: Always check existing port usage (`docker ps`) before binding. Never assume standard ports are free on shared servers.

### 10. git clone into non-empty directory
**What happened**: `git clone <url>` without `.` created a subdirectory instead of cloning into current dir. User then couldn't find `.env.example`.
**Rule**: Use `git clone <url> .` to clone into current directory. Verify with `ls` after clone.

### 11. Read tool required before Write on existing files
**What happened**: Claude Code Write tool rejects writes to files that haven't been read first in the session, even for new content.
**Rule**: Always Read a file before attempting to Write/Edit it, even if you know the contents.

### 12. Context window exhaustion during large builds
**What happened**: Building 54+ files in a single session repeatedly hit context limits, requiring session continuations.
**Rule**: For large multi-file builds, create files in parallel batches and commit frequently. Use the plan file to track progress across sessions.

### 13. Test assertion against wrong validation layer
**What happened**: Wrote a test expecting a "gender" error message, but the tracking_id regex `TGMA-(WT|ST|DL)-(M|F)-(\d{3,4})` rejected the ID first (before gender validation ran). Test failed with "Invalid format" instead of "gender".
**Fix**: Updated test to assert `error is not None` instead of checking for a specific error substring.
**Rule**: When writing validation tests, understand the validation order. If field A is validated before field B, a test for field B rejection must use input that passes field A first. The tracking_id regex enforces gender (M|F) at the format level — there's no separate gender rejection path for IDs with invalid gender codes.

### 14. ETL script as CLI-only, not importable
**What happened**: Original `etl/kobo_sync.py` was a CLI-only script with `main()` that created its own app context. Couldn't be called from a Flask route without duplicating logic.
**Fix**: Refactored into an importable `run_sync(app, triggered_by, full_sync)` function that accepts an app instance. CLI `main()` just calls `run_sync()`. Route handler also calls `run_sync()`.
**Rule**: Always design ETL/sync scripts as importable modules with a public function. The CLI entry point should be a thin wrapper. This lets both UI routes and CLI share the same logic without duplication.

### 15. Upsert clobbering existing data with NULLs
**What happened**: When a KoboToolbox form has optional sections (e.g., anthropometrics left empty), the sync would overwrite existing non-NULL values with NULL on re-sync.
**Fix**: Upsert logic only sets attributes where the new value `is not None`. Also skip creating related records (anthro, lifestyle, env) entirely if ALL their fields are NULL.
**Rule**: In upsert logic, never overwrite existing data with NULL. Only update fields that have actual values. Check `any(v is not None for v in data.values())` before creating related records for optional sections.

### 16. Importing from parent directory in route handlers
**What happened**: The kobo route needed to import `run_sync` from `etl/kobo_sync.py` which lives outside the Flask app package.
**Fix**: Used `sys.path.insert(0, ...)` inside the route function to add the project root, then imported `from etl.kobo_sync import run_sync`.
**Rule**: When importing from outside the app package (e.g., `etl/`), add project root to `sys.path` at import time. This is acceptable for a LAN-only deployment. For cleaner architecture, consider moving shared logic into the app package.

### 17. New model table missing on deployed server (500 Internal Server Error)
**What happened**: Added `KoboSyncLog` model with `kobo_sync_log` table locally, committed, and pushed. On the deployed server, the Docker container was rebuilt but `db.create_all()` was not re-run — so the table didn't exist. Visiting `/kobo/` caused a 500 error because SQLAlchemy tried to query a non-existent table.
**Fix**: Added `_ensure_table()` helper in the kobo route that uses `sqlalchemy.inspect(db.engine).has_table('kobo_sync_log')` and creates it on-the-fly if missing. This makes the route self-healing on first access after deploy.
**Rule**: When adding a new model/table, always handle the case where the deployed DB doesn't have it yet. Either: (a) add auto-create logic in the route, (b) use Flask-Migrate (`flask db migrate && flask db upgrade`), or (c) document that `init_db.py` must be re-run. Never assume `db.create_all()` has run for new tables on the server.

### 18. Nested app context crash when calling ETL from Flask route
**What happened**: `run_sync()` used `with app.app_context():` unconditionally. When called from a Flask route (which already has an app context), this created a nested context. On exit, it popped the outer request context too, causing the returned `sync_log` object to become detached from the session → 500 Internal Server Error.
**Fix**: Split into `_do_sync()` (core logic, no context management) and `run_sync()` (wrapper that checks `flask.has_app_context()` — if already in context, calls directly; if CLI, pushes a new one). Also wrapped the route handler in try/except so errors flash a message instead of returning a raw 500.
**Rule**: Never unconditionally push `app.app_context()` in functions that may be called from both CLI and Flask routes. Use `flask.has_app_context()` to detect if a context already exists. Always wrap route-triggered operations in try/except to show user-friendly error messages.

### 19. `Query.delete()` bypasses ORM cascade rules
**What happened**: When writing the `wipe_data.py` reset script, the obvious-looking `Participant.query.delete()` would have left orphaned rows in `samples`, `hormone_results`, `anthropometrics`, etc. SQLAlchemy `Query.delete()` is a bulk SQL DELETE that does NOT trigger Python-side cascade rules defined on relationships.
**Fix**: Iterate `Participant.query.all()` and call `db.session.delete(p)` per row so the `cascade='all, delete-orphan'` paths in `app/models/participant.py:80-95` actually fire.
**Rule**: Whenever a model has `cascade='all, delete-orphan'` relationships, never use `Query.delete()` to remove rows — always iterate and `db.session.delete()` per object. `Query.delete()` is fine only for tables with no Python-side cascades (e.g., `IdAllocation`, `KoboSyncLog`).

### 20. URL path segments joined into filesystem paths without validation
**What happened**: When building the Document Vault upload route, `tracking_id` flows from a URL path parameter straight into `os.path.join(UPLOAD_FOLDER, 'participants', tracking_id, doc_type)`. Even though Flask route converters give you a string, nothing stops a caller from sending a percent-encoded `../` sequence that `os.path.join` will happily normalize into an escape from the uploads volume.
**Fix**: Call `validate_tracking_id()` from `app/utils/helpers.py` at the top of every route that uses a `tracking_id` to build a filesystem path. The regex `^TGMA-(WT|ST|DL)-(M|F)-(\d{3,4})$` rejects anything with slashes, dots, or percent-encoded escapes.
**Rule**: Never concatenate a URL path parameter into a filesystem path without validating it against a strict regex first. Apply this to *any* user-controlled string used in `os.path.join`/`open`/`send_file`, not just tracking IDs. Also cascade-delete orphaned files from disk in wipe scripts — DB cascade does not clean up the filesystem (see `scripts/wipe_data.py` which `shutil.rmtree`s the `participants/` upload subtree).

### 21. `scripts/` not in `sys.path` when run inside Docker container
**What happened**: `docker compose exec web python scripts/wipe_data.py` raised `ModuleNotFoundError: No module named 'app'` because the script lives at `/app/scripts/wipe_data.py` and Python looks for `app` relative to the script directory (`/app/scripts/`), not the working directory (`/app/`).
**Fix**: Add `sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))` at the top of the script — identical to the pattern in `scripts/init_db.py:14-15`.
**Rule**: Every script in `scripts/` that imports from the `app` package must include this `sys.path.insert` line. Use `scripts/init_db.py` as the reference template.

## Architecture Rules

### DO
- Use `tracking_id` (VARCHAR 20) as primary key for all participant-related tables
- Keep sample types as `stool` internally (display as "Fecal" in UI)
- Use `saliva_cortisol` for the new cortisol saliva sample type (suffix: COR)
- Use green color palette: #2D6A4F (primary), #52B788 (secondary), #95D5B2, #B7E4C7
- Use `tgma-card` class for cards, `stat-card` for stat cards, `btn-tgma` for buttons
- Use Satoshi font family (woff2 files in `app/static/vendor/fonts/`)
- All vendor libraries must be local files (no CDN — LAN-only server has no internet)
- DataTables for server-side paginated tables (participants list)
- Chart.js for dashboard visualizations
- Design ETL scripts as importable modules with a public function + thin CLI wrapper
- In upsert logic, only overwrite with non-NULL values (preserve existing data)
- Skip creating related records if ALL fields are NULL (optional empty sections)
- Gate admin features (sync, ID allocation, document uploads/deletes) with `@role_required('pi', 'co_pi', 'bioinformatician')`
- Always call `validate_tracking_id()` from `app/utils/helpers.py` before joining a `tracking_id` into a filesystem path — defense against path traversal via URL segments
- Test with `conda run -n base python -m pytest tests/ -v` on Windows dev machine
- Commit with `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`

### DO NOT
- Do NOT add CDN links — all assets must be local (LAN-only, no internet on server)
- Do NOT use surrogate keys for participant tables — `tracking_id` is the PK
- Do NOT store passwords in plain text anywhere (not even in comments)
- Do NOT use `git push --force` on main
- Do NOT modify `.env` on the server without checking current values first
- Do NOT run `docker compose down -v` unless explicitly asked (destroys all data)
- Do NOT add WeasyPrint or heavy system dependencies to Dockerfile unless specifically needed
- Do NOT assume Python 3.x is default on Windows — use `conda run -n base python`
- Do NOT use multiline strings in `conda run -n base python -c` on Windows
- Do NOT overwrite existing DB values with NULL during upsert/sync operations
- Do NOT auto-sync KoboToolbox on a schedule — manual "Sync Now" button only (PI controls when)
- Do NOT accept KoboToolbox submissions missing critical fields (tracking_id, full_name, gender, district) — reject and log the reason
- Do NOT write test assertions against a specific error message when multiple validation layers exist — assert `error is not None` unless you're sure which layer fires first
- Do NOT push `app.app_context()` unconditionally in functions called from both routes and CLI — always check `flask.has_app_context()` first
- Do NOT let route handler exceptions bubble up as raw 500s — always wrap in try/except and flash a user-friendly message

## CSS Theme Reference
- **Background**: `--tgma-bg: #F5F3EF`
- **Card**: `--tgma-card: #FFFFFF` with `--tgma-border: #E8E5DF`
- **Primary accent**: `--tgma-accent: #2D6A4F` / hover: `#245A42`
- **Light green**: `--tgma-accent-light: #D8F3DC`
- **Greens**: `--tgma-green-500: #52B788`, `--tgma-green-300: #95D5B2`, `--tgma-green-200: #B7E4C7`
- **Text**: `--tgma-text: #1B1B1B`, muted: `--tgma-muted: #6B7280`
- **Radius**: `--tgma-radius: 14px`, small: `10px`
- **Sidebar width**: `260px`
- **Responsive**: sidebar hidden below 768px
- **Print**: sidebar, top-bar, buttons, tabs hidden

## Study Parameters (config.py)
| Parameter | Value |
|-----------|-------|
| TARGET_ENROLLMENT | 440 |
| TARGET_SAMPLES_YEAR1 | 160 |
| TARGET_SEQUENCING | 160 |
| DISTRICT_TARGETS | WT: 200, ST: 100, DL: 100 |
| LIFESTYLE_GROUPS | AT, AP, SDT, SP (100 each) |
| SEQUENCING_BATCH_SIZE | 32 |
| TOTAL_BATCHES_YEAR1 | 5 |
| GPS bounds | Lat: 22.9–24.5, Lon: 91.1–92.3 |

## Users
| Username | Role | Real Person | Default Password |
|----------|------|-------------|-----------------|
| surajit_b | pi | Dr. Surajit Bhattacharjee | SurajitPI@2026 |
| shib_d | co_pi | Dr. Shib Sekhar Datta | ShibCoPI@2026 |
| sanchari_p | bioinformatician | Miss. Sanchari Pal (Project Scholar) | SanchariTGMA@2026 |
| argajit_s | bioinformatician | Mr. Argajit Sarkar (Project Scholar) | ArgajitTGMA@2026 |
| field_sup | field_supervisor | Field Supervisor | FieldTGMA@2026 |

## KoboToolbox Sync Strategy
- **Trigger**: Manual only — PI/Co-PI/Bioinformatician clicks "Sync Now" in UI (`/kobo`)
- **Modes**: Incremental (since last sync) or Full (re-fetch everything)
- **Critical fields (REJECT if missing)**: `tracking_id`, `full_name`, `gender`, `district`
- **Optional fields (accept as NULL)**: age, DOB, GPS, anthropometrics, lifestyle, environment, menstrual
- **Idempotent**: Upserts by `tracking_id` — syncing same submission twice updates existing record
- **NULL-safe**: Only overwrites with non-NULL values; empty optional sections don't create empty related records
- **GPS out-of-bounds**: WARNING logged but submission accepted (not rejected)
- **Gender/District fallback**: If missing from form fields, derived from tracking_id pattern (TGMA-WT-F-0001 → district=WT, gender=F)
- **Sync log**: Every run stored in `kobo_sync_log` table with counts + per-submission JSON details
- **API**: KoboToolbox REST API v2, paginated (100/page), sorted by `_submission_time`
- **State file**: `etl/.kobo_sync_state.json` stores last sync timestamp for incremental mode
- **KoboToolbox behavior**: Only fully submitted forms reach the server. Drafts stay on the phone. Field workers may skip optional sections.

## Git History (as of April 2026)
```
4c72f01 Fix wipe_data.py ModuleNotFoundError: add project root to sys.path
24171ac Add per-participant Document Vault for scanned forms and photos
0615b3e Add participant edit/delete UI, demo-data wipe script, and batched label-kit access
6a08b2f Add Label Kit Excel generator for Seznik thermal printer workflow
8b01ceb Fix DataTables sort icon rendering and participants table column widths
4756173 Add participant stat cards and fix KoboSync placeholder credential error
45479c2 Add ngrok public tunnel, 5-user upsert, and label sheet for field worker workflow
c27ba99 Update CLAUDE.md: mistakes #17-18, ETL split pattern, git history
0173610 Fix Sync Now 500 error: nested app context + unhandled exception
bf78019 Fix KoboToolbox sync 500 error: auto-create missing table on first access
6de0359 Update CLAUDE.md with KoboToolbox sync docs, 4 new mistakes, and architecture rules
ebaaa28 Add KoboToolbox sync with manual UI trigger, validation, and sync log
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/argajitsarkr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
