## sass-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

FounderStack — production-ready SaaS boilerplate. Django 5.1 + DRF + Channels (Python 3.12) + Next.js 15 / React 19 (pnpm). Multi-tenant with RBAC, JWT auth, Stripe, Celery, Postgres, Redis, optional R2/S3 uploads. Tier system: T1 Core → T2 Standard → T3 Premium (see `TIER.md`).

## Live project context

Always treat these as source of truth — do not duplicate their content here:

- @TIER.md — active tier and feature flags for this checkout
- @docs/PROGRESS.md — current phase, completed modules, known limitations, env vars added
- @docs/architecture.md — system diagram, Django app responsibilities, frontend layout

## Environment

- Platform: Windows 11, PowerShell default shell. Use PowerShell syntax (`$env:VAR`, backticks, no `&&`). Bash tool also available for POSIX.
- Backend runs in Docker via `infra/docker-compose.yml` (services: `db`, `redis`, `backend`, `worker`, `beat`). Run from `infra/`.
- Frontend runs locally with pnpm. Run from `frontend/`.
- Required env: copy `.env.example` → `backend/.env`; copy `frontend/.env.example` → `frontend/.env.local`.

## Commands

### Backend (run inside the `backend` container or with activated venv)

| Task | Command |
|------|---------|
| Run tests | `pytest` (settings: `config.settings.test`, configured in `pyproject.toml`) |
| Run single test | `pytest -k "test_name"` or `pytest path/to/test_file.py::TestClass::test_method` |
| Coverage | `pytest --cov` (fails under 70%) |
| Lint | `ruff check .` |
| Format | `ruff format .` and `black .` (both line-length 100) |
| Typecheck | `mypy .` (uses `mypy_django_plugin`, settings module `config.settings.development`) |
| Migrate | `python manage.py migrate` |
| Make migrations | `python manage.py makemigrations` |
| Sync Stripe plans | `python manage.py sync_stripe_plans` |
| Seed demo data | `python manage.py seed_demo_data` |

### Frontend (from `frontend/`)

| Task | Command |
|------|---------|
| Dev server | `pnpm dev` |
| Typecheck | `pnpm type-check` (== `tsc --noEmit`) — what CI runs |
| Lint | `pnpm lint` (eslint via `eslint-config-next`) |
| Build | `pnpm build` |
| Regenerate API types | `pnpm gen:api` (requires backend running on `localhost:8000`) |
| Build email templates | `pnpm email:export` (compiles React Email → Django HTML templates in `backend/templates/emails/`) |
| Preview emails | `pnpm email:preview` |

### Infra

```powershell
cd infra
docker compose up --build      # start db, redis, backend, worker, beat
docker compose exec backend python manage.py createsuperuser
```

## Architecture rules

- **Multi-tenancy:** `apps/core/models.py::TenantAwareModel` carries `organization` FK. Views that return tenant data must inherit `TenantScopedViewMixin` or filter by `request.organization` (set by `apps.organizations.middleware.CurrentOrganizationMiddleware` from `X-Organization-Id` header or `current_org_id` cookie).
- **RBAC:** roles are numeric `IntegerChoices` (VIEWER=1, MEMBER=2, ADMIN=3, OWNER=4) so `>=` comparisons work. Gate views with `apps.core.permissions.RequiresRole`.
- **Feature flags:** two systems coexist —
  1. **Tier gate** — static dict in `backend/config/features.py` + `TIER.md`. Wrap views with `@feature_required("flag_name")` from `apps.core.decorators`. Frontend reads via `useFeatureFlags()`.
  2. **Per-org runtime flags** — `apps.feature_flags` Redis-cached (5s TTL), admin-toggleable.
- **API surface:** all routes mounted at `/api/v1/` via `apps/api/v1/urls.py`. New apps must register here. OpenAPI at `/api/v1/schema/`, Swagger at `/api/v1/docs/`.
- **Auth:** refresh token in httpOnly cookie, access token in memory. `apps.accounts.auth` handles register/verify/login/refresh/logout/password-reset/2fa/oauth. Frontend never reads the refresh token.
- **Stripe webhooks:** idempotency via `ProcessedStripeEvent` table — survives Redis flush. Use `event["key"]` dict-style access (compatible with both live stripe-python and fixture replay).
- **Migration order matters:** `accounts` → `organizations.0001` → `billing.0001` → `organizations.0002_organization_plan` (adds Org.plan FK). Don't reorder.
- **WebSockets:** ASGI via Uvicorn (not gunicorn). `config/asgi.py` `ProtocolTypeRouter` routes `ws/` to `apps.notifications.routing` with `JWTAuthMiddleware` from `apps.core.channel_auth`.
- **Uploads:** R2 presigned POST — bytes never proxy through Django. `apps.uploads.services.generate_presigned_post()` + `ConfirmUploadView`.

## Adding a new module

1. `python manage.py startapp mymodule` inside `backend/apps/`
2. Append to `LOCAL_APPS` in `backend/config/settings/base.py`
3. Add serializers/views/urls; register in `apps/api/v1/urls.py`
4. Add TanStack Query hooks under `frontend/hooks/use-<resource>.ts`
5. If tenant-scoped, inherit `TenantAwareModel` and `TenantScopedViewMixin`
6. Add tests under `apps/<module>/tests/`

## Workflow conventions

- **Do not create one commit per task.** Implement the full set of requested changes, let the user review, then commit at the end of the session unless explicitly told otherwise.
- **Check `docs/PROGRESS.md` at session start** to understand current phase and what's already done. Update it when a module is completed.
- Branch naming: feature branches off `main`. Current dev branch is `tire-2`.
- CI runs pytest (backend) and `tsc --noEmit` (frontend) on push to `main`/`develop` and PRs to `main`. Lint is **not** in CI yet — run `ruff check` and `pnpm lint` locally.

## Gotchas

- `pnpm email:export` must be run after editing anything in `frontend/emails/` — otherwise Django will render stale HTML templates from `backend/templates/emails/`.
- `USE_R2=false` (default in dev) stores uploads locally; presign flow still runs but signs a local URL.
- Test settings use eager Celery (`CELERY_TASK_ALWAYS_EAGER = True`) — `task.delay()` runs synchronously in tests. Don't assert on Celery queue state.
- `AUTH_COOKIE_SECURE=false` in dev; production settings force `True`.
- Coverage gate is 70% (`pyproject.toml` `fail_under`).

---
> Source: [numanijaz119/Sass-Template](https://github.com/numanijaz119/Sass-Template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
