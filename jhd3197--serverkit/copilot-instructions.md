## serverkit

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

ServerKit is a server control panel for managing web applications, databases, Docker containers, and security on VPS/dedicated servers. Flask backend (Python 3.11+), React frontend (Vite + SCSS), SQLite/PostgreSQL database, real-time updates via Socket.IO.

## Development Commands

```bash
# Backend (launcher default port 47927, hot-reload)
cd backend && python -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
python run.py

# Frontend (launcher default port 41921, Vite HMR)
cd frontend && npm install && npm run dev

# Both at once (Linux/WSL)
./dev.sh

# Frontend lint
cd frontend && npm run lint

# Frontend production build
cd frontend && npm run build

# Backend tests
cd backend && pytest
cd backend && pytest --cov=app

# Docker
docker compose -f docker-compose.dev.yml up --build
```

Default dev credentials: `admin` / `admin`

## Architecture

### Backend (`backend/`)

Flask app factory in `app/__init__.py` using `create_app()`. Three-layer architecture:

- **`app/api/`** — Flask Blueprints, one file per feature (90+ files). All routes prefixed `/api/v1/`. JWT-protected via `@jwt_required()`.
- **`app/services/`** — Business logic (165+ files). Services are stateless modules called by API routes. Heavy lifting (shell commands, Docker API, file operations) happens here.
- **`app/models/`** — SQLAlchemy ORM models (80+ files). Schema is managed by Alembic migrations in `backend/migrations/versions/` — add a migration for every schema change; the app does not call `db.create_all()`.

Other backend components:
- `app/sockets.py` — Socket.IO event handlers for real-time metrics, logs, terminal
- `app/agent_gateway.py` — Multi-server agent communication
- `app/middleware/security.py` — Security headers middleware
- `config.py` — Environment-based config (development/production/testing)
- `run.py` — Entry point

### Frontend (`frontend/src/`)

React 18 SPA with client-side routing:

- **`pages/`** — Route-level components (60+ files). Each maps to a route in `App.jsx`.
- **`components/`** — Reusable UI components shared across pages.
- **`contexts/`** — React Context providers: `AuthContext` (JWT auth + token refresh), `ThemeContext`, `ToastContext`, `ResourceTierContext` (feature gating).
- **`services/api.js`** — Centralized `ApiService` class handling all HTTP requests, token management, and auto-refresh.
- **`hooks/`** — Custom React hooks for reusable logic.
- **`styles/`** — SCSS stylesheets with design system variables. Main entry is `main.scss`. Page-specific styles in `styles/pages/`.
- **`layouts/`** — `DashboardLayout` wraps authenticated pages (sidebar + header).

Route guards: `PrivateRoute` (auth check), `PublicRoute` (redirect if logged in), `SetupRoute` (redirect to `/setup` if not configured).

### Request Flow

Browser → Nginx (`:80`/`:443`) → proxy_pass to Docker containers (`:8001-8999`) for managed apps, or to Flask (`:5000`) for the panel API. The 404 handler in Flask serves `index.html` for SPA client-side routing; API routes return JSON errors.

### Production Build

The Dockerfile is multi-stage: Node 20 builds frontend, Python 3.11 serves everything via Gunicorn with a single plain threaded worker (`--workers 1 --threads 100`, no `--worker-class` — see the single-worker note above). Built frontend is served from Flask's static folder.

## Platform & Distro Awareness

ServerKit deploys on Linux (bare metal, VPS, or Docker). Development may happen on Windows/macOS.

- **Service layer is Linux-only** — nginx, systemctl, apt/dnf, PHP-FPM, etc. are inherently Linux. No need to abstract these for Windows.
- **Platform-agnostic code** (config management, storage, API layer) should guard Unix-only calls like `os.chmod` with `if os.name != 'nt'` so the dev server can run locally on any OS.
- **Distro differences matter** — use `backend/app/utils/system.py` helpers (`get_package_manager`, `is_package_installed`, `install_package`) instead of calling `apt`/`dpkg`/`dnf` directly. Not all targets are Debian-based.

## Code Style

### Python
- PEP 8, type hints where helpful
- Service functions are standalone (no classes unless stateful)
- Consistent JSON error responses: `{'error': 'message'}, status_code`

### React/JavaScript
- Functional components with hooks only
- PascalCase for components (`Sidebar.jsx`), camelCase for everything else
- SCSS for styling — use existing design system variables (`$bg-card`, `$primary-color`, `$spacing-md`, etc.) and BEM-like naming (`.block__element--modifier`)
- Context API for global state; props drilling is fine for 2-3 levels
- No inline styles; no Tailwind/CSS-in-JS

### Diffs & Commits
- One logical change per commit
- Minimal, focused diffs — don't silently refactor surrounding code
- Branch naming: `feature/`, `fix/`, `docs/`, `refactor/` prefixes
- **Pre-commit hook**: a shared git hook (`.githooks/pre-commit`) lints staged
  frontend JS/JSX and blocks the commit on ESLint **errors** (warnings allowed;
  backend/SCSS-only commits are skipped). Enable once per clone:
  `git config core.hooksPath .githooks` (or `sh frontend/setup-hooks.sh`). Keep
  the repo at **0 ESLint errors**.

## Styling Standard

**SCSS only.** ServerKit standardizes on its internal SCSS design system. Do not
add Tailwind utility classes, CSS-in-JS, or new inline styles.

- **Forbidden**: Tailwind utility strings (`flex gap-3 px-4`, `text-sm`,
  `bg-card`, `border-border`, `rounded-full`, etc.), `tailwindcss`/`tailwind-merge`
  imports, and CSS-in-JS. Inline `style={{ ... }}` is allowed **only** for true
  dynamic values (a width computed from a prop, chart dimensions).
- **Class namespaces**:
  - `.sk-*` — redesign "infra console" design-system primitives.
  - Legacy BEM (`.page-container`, `.card`, `.modal-*`, `.form-*`,
    `.empty-state__title`) — allowed during migration; deprecate gradually.
  - No utility-class strings.
- **Shared primitives** (reuse before rewrite):
  - Buttons — `styles/components/_buttons.scss` (`.btn`, `.btn-primary`,
    `.btn-danger`, `.btn-ghost`, `.btn-sm`, `.btn-icon`).
  - Cards — `styles/components/_cards.scss` (`.card`, `.card-header`,
    `.stat-strip`).
  - Modals — `styles/components/_modals.scss` (`.modal-overlay`, `.modal`,
    `.modal-lg`, `.modal-header`, `.modal-body`, `.modal-actions`).
  - Forms — `styles/components/_forms.scss` (`.form-group`, `.form-field`,
    `.form-row`, `.error-message`); prefer the `FormField` component.
  - Tables — `.sk-dtable` in `styles/components/_design-system.scss`; prefer the
    `components/ds/DataTable.jsx` primitive.
  - Pills/badges — `.sk-pill`, `.sk-state`, `.sk-tag`.
  - Tokens — `styles/_variables.scss`, `styles/_theme-variables.scss`.
- **Shared utilities** (reuse before rewrite):
  - `utils/formatBytes.js` — byte/size formatting (no local `formatBytes`/
    `formatSize`/`formatMemory`).
  - `utils/time.js` — `timeAgo` (compact) and `formatRelativeTime` (verbose).
  - `utils/clipboard.js` + `hooks/useClipboard.js` — copy-to-clipboard with
    toast feedback (no raw `navigator.clipboard.writeText` call sites).
- **Fonts**: `IBM Plex Sans` / `IBM Plex Mono` (the established SCSS stack).

**Migration status**: Tailwind removal is in progress (see
`docs/plans/05_FRONTEND_UX_TAILWIND_CLEANUP_PLAN.md`). Do not introduce new Tailwind
usage; convert any you touch to the SCSS equivalents above.

## Adding a New Feature (Full Stack)

1. **Model**: Add SQLAlchemy model in `backend/app/models/`
2. **Service**: Add business logic in `backend/app/services/`
3. **API**: Create Blueprint in `backend/app/api/`, register it in `app/__init__.py` with `url_prefix='/api/v1/<feature>'`
4. **Frontend API**: Add methods to `ApiService` in `frontend/src/services/api.js`
5. **Page**: Create page component in `frontend/src/pages/`, add route in `App.jsx`
6. **Styles**: Add SCSS file in `frontend/src/styles/pages/`, import in `main.scss`

## Key Environment Variables

| Variable | Purpose |
|----------|---------|
| `SECRET_KEY` | Flask session signing |
| `JWT_SECRET_KEY` | JWT token signing |
| `DATABASE_URL` | DB connection string (`sqlite:///...` or PostgreSQL) |
| `CORS_ORIGINS` | Comma-separated allowed origins |
| `FLASK_ENV` | `development` or `production` |

## Builtin Extensions

Extensions live in `builtin-extensions/<name>/` with a `plugin.json` manifest
(required keys: `name`, `display_name`, `version`; `entry_point` as
`module:attr`) and a `backend/` package (loaded as `app.plugins.<name>`).
API-key auth: guard routes with `@auth_required()` / `@admin_required` from
`app/middleware/rbac.py` — bare flask `@jwt_required()` rejects `X-API-Key`
callers.

- `serverkit-localkit` — API surface for the LocalKit desktop app, mounted at
  `/api/v1/localkit` (`GET /pair`, `GET|POST /sites`, `POST /push/code`,
  `POST /push/db`, `GET /pull/db`). It is the API-key-friendly surface for
  WordPress automation (the hub `GET /api/v1/wordpress/sites` is JWT-only).
  Reuses the `serverkit-wordpress` service layer (imported cross-extension via
  `importlib` as `app.plugins.serverkit-wordpress.wordpress_service`) and
  `DatabaseSyncService.import_to_container`/`export_from_container` for DB
  push/pull; wp-content arrives as a tar.gz and is installed via `docker cp`.

---
> Source: [jhd3197/ServerKit](https://github.com/jhd3197/ServerKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
