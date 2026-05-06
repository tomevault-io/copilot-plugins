## deployment-hardening

> Production deployment requirements — Docker, DigitalOcean, Vercel, CORS, pool, auth, observability


# Deployment Hardening

Deviations are bugs, not preferences.

## Deployment targets

Primary: **DigitalOcean** (or equivalent API host for the backend) + **Vercel** (frontend) + **Supabase** (auth + Postgres).

Optional: `docker-compose.yml` runs **Redis + backend** locally for container smoke checks. Database is always Supabase (local or hosted). It is not the production topology.

There are three environments: `development`, `test`, `production`. There is no staging.

## Backend host (production)

- Ship the same `backend/Dockerfile` image; the host sets listen `PORT` (e.g. `${PORT:-8000}` in CMD).
- Postgres uses Supavisor **session pooler** (port 5432, IPv4) in production. Direct connections (`db.xxx.supabase.co`) resolve to IPv6 only, which DO App Platform cannot route. Session mode preserves per-connection state and supports asyncpg prepared statements. Do NOT use the transaction pooler (port 6543) - asyncpg prepared statements are incompatible with transaction pooling. `PUBLIC_DB_USER=postgres.<project-ref>`, `PUBLIC_DB_HOST=aws-0-<region>.pooler.supabase.com`, `PUBLIC_DB_PORT=5432`, `PUBLIC_DB_SSL_MODE=require` are committed in `backend/.env.production`. `PRIVATE_DB_PASSWORD` comes from secrets. A single `PRIVATE_DATABASE_URL` secret is still supported; config normalizes it to `postgresql+asyncpg://`.
- `PRIVATE_JWT_SECRET` must be the Supabase project's JWT secret — not a random value.
- `PUBLIC_CORS_ORIGINS` must include all stable Vercel domains. `PUBLIC_CORS_ORIGIN_REGEX` can match preview deploys.
- `PUBLIC_FRONTEND_URL` needed for Xero OAuth redirects.
- Health check path used in CI smoke tests: `GET /api/beta/shared/health`.

## Vercel

- **Single** root `vercel.json` at repo root controls the Vercel project (build `cd frontend && pnpm install --frozen-lockfile && pnpm run build`, output, CSP, `git.deploymentEnabled`). No second `frontend/vercel.json` (avoids drift). `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID` are **not** committed in `vercel.json`; use GitHub Environment secrets or gitignored `.vercel/project.json` after `vercel link`.
- Output: `frontend/dist`. SPA rewrite: all routes → `index.html`.
- Three `VITE_*` env vars are build-time (baked into JS). Source of truth for production: committed `frontend/.env.production` (loaded by Vite when `pnpm run build` runs in mode production). Changing requires redeploy.
- **Git vs Actions:** Repo sets `git.deploymentEnabled` with the production branch name (e.g. `main: false`) so Vercel’s [GitHub integration](https://vercel.com/docs/git/vercel-for-github) does not auto-deploy that branch while GitHub Actions runs `pull` + `build --prod --yes` + `deploy --prebuilt --prod --yes` (see [GitHub Actions + Vercel](https://vercel.com/docs/git/vercel-for-github#using-github-actions)). The key must match the **Production Branch** in Vercel project settings. Pin CLI with `npx vercel@50` under pixi. Deploy job attaches `--meta` (`githubCommitSha`, `githubCommitRef`, `githubRepo`, `githubActionsRunId`) for filtering in `vercel list --meta`. Environment secrets: `frontend_vercel_deployment` with `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`.
- Optional: Vercel can emit [`repository_dispatch`](https://vercel.com/docs/git/vercel-for-github#repository-dispatch-events) to GitHub (e.g. `vercel.deployment.success`) for post-deploy E2E; prefer that over `deployment_status` if you disable deployment notifications in Vercel Git settings.
- Security headers set in `vercel.json`: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`.
- Immutable cache headers on `/assets/`.
- Build-time env template: `frontend/.env.example`.

## Docker

- Two-stage build: `builder` (compilers, uv sync) → `runtime` (venv only, no headers)
- uv is pinned via `COPY --from=ghcr.io/astral-sh/uv:0.7` in the builder stage
- Non-root: `appuser:appgroup` uid/gid 1001, `USER appuser` before `CMD`
- `ENV PYTHONUNBUFFERED=1 PYTHONDONTWRITEBYTECODE=1` in runtime stage — required for log streaming
- `CMD` uses gunicorn with `uvicorn.workers.UvicornWorker` for process management. Pass `--worker-tmp-dir /dev/shm` - DigitalOcean App Platform's container runtime can break gunicorn's default worker temp dir ([Dockerfile build reference](https://docs.digitalocean.com/products/app-platform/reference/dockerfile/)). No `uv run` wrapper in production.
- `devtools/` never copied into image

## CI/CD

- GitHub Actions **CI** (`.github/workflows/ci.yml`): standalone on **push to `dev`**, all **pull requests** to `main`/`dev`, **`workflow_dispatch`**, and **`workflow_call`** from **CD**. On **`main` pushes** there is **no** standalone CI - only the CD workflow runs and calls `ci.yml` as the CI gate (avoids duplicate test runs).
- **`dorny/paths-filter@v4`** drives **all four CI jobs**. Backend lint and test run when `backend/**` or `supabase/**` changed. Frontend lint runs when `frontend/**` changed. Frontend test runs when backend, supabase, **or** frontend changed (cross-stack regressions). If nothing in those dirs changed, all jobs skip.
- Path filters: backend includes `backend/**` and `supabase/**`; frontend includes `frontend/**`. Pull-request runs need `pull-requests: read` on the `changes` job; push runs use checkout + git-based detection (`base: ${{ github.ref }}` for same-branch pushes).
- **CD** (`.github/workflows/cd.yml`) triggers on push to `main` when `backend/**`, `frontend/**`, `.do/**`, or either workflow file changes. A **`changes`** job outputs **`backend_build`** (image rebuild) vs **`backend_deploy`** (App Platform apply); **`ci`** must pass; **`build-backend`** runs only when `backend_build` matches; **`deploy-backend`** runs when `backend_deploy` matches (spec-only pushes skip the Docker build and use image tag `latest`). **`workflow_dispatch`** sets all three flags true (full gate + full deploy).
- Backend deploy: Docker build → push to `ghcr.io/<owner>/sku-ops-backend` (short SHA + `latest`) → `envsubst` renders `.do/app.yaml` with `PRIVATE_*` + `GHCR_REGISTRY_CREDS` secrets from the **`backend_digital_ocean_deployment`** environment → `doctl apps update --spec` → `doctl apps create-deployment --wait`. Environment secrets: `DIGITALOCEAN_ACCESS_TOKEN`, `DIGITALOCEAN_APP_ID`, `GHCR_REGISTRY_CREDS`, and every `PRIVATE_*` key listed in the spec template (see below).
- App Platform spec: `.do/app.yaml` is a **template** (placeholders for `IMAGE_TAG`, `PRIVATE_*`, and `GHCR_REGISTRY_CREDS`). `GHCR_REGISTRY_CREDS` (`username:PAT_with_read_packages`) is injected into `image.registry_credentials` on every deploy so DO can pull the private GHCR image; omitting it from a `doctl apps update --spec` clears previously saved credentials. All **non-secrets** are committed in `backend/.env` / `backend/.env.production` and baked into the image; do not duplicate them in the spec. `PUBLIC_REDIS_URL` comes from the Valkey **database binding** (`${valkey-cache.DATABASE_URL}`). Bootstrapping with `do_apps_create_ghcr.py` requires a **rendered** spec or manual substitution for `$IMAGE_TAG` / secrets.
- Frontend deploy: see **Vercel** above (prebuilt prod deploy via Actions + Environment secrets).

## CORS

`_enforce_cors()` in `config.py` hard-crashes at startup if `PUBLIC_CORS_ORIGINS` is `"*"` or empty in production. Not a warning.
`PUBLIC_CORS_ORIGIN_REGEX` is applied in addition to `PUBLIC_CORS_ORIGINS` — useful for Vercel preview deploy URLs.

## Connection pool

`PUBLIC_PG_POOL_MAX` per worker × `PUBLIC_WORKERS` = total Postgres connections. Size accordingly. `PUBLIC_PG_ACQUIRE_TIMEOUT` (default 10s) is the starvation circuit-breaker — requests 503 rather than queue. `PUBLIC_PG_COMMAND_TIMEOUT` (default 30s) kills runaway queries.

## Auth

- Tokens without `organization_id` → 401/4001 at transport layer in production
- `PRIVATE_JWT_SECRET` must be set and not the dev default in production — startup crashes otherwise
- For Supabase: `PRIVATE_JWT_SECRET` must be the Supabase project's JWT secret (Dashboard > Settings > API)
- Provider is auto-selected: supabase in production, internal in dev/test. Claim extraction lives only in `shared/api/auth_provider.py`

## Health probes

- `/api/health` — liveness, never blocks on DB, must complete < 100ms
- `/api/ready` — readiness, checks DB pool. Load balancers gate traffic on this, not `/api/health`

## Observability

- `PRIVATE_SENTRY_DSN` should be set in production. Do not suppress exceptions to reduce Sentry noise — fix the underlying issue.
- Application logs are JSON (configured in `shared/infrastructure/logging_config.py`). Every log line within a request carries `request_id` from the `X-Request-ID` propagation middleware.

## Required production env vars (backend)

**Non-secrets (committed, baked into the Docker image):** see `backend/.env.production` (`PUBLIC_DB_*`, `PUBLIC_SUPABASE_*`, `PUBLIC_CORS_ORIGINS`, `PUBLIC_FRONTEND_URL`, `PUBLIC_WORKERS`, `PUBLIC_XERO_*`, etc.).

**Secrets (GitHub Environment `backend_digital_ocean_deployment` → rendered into `.do/app.yaml`, not the DO dashboard):**

```
PRIVATE_DB_PASSWORD=<supabase-database-password>
PRIVATE_JWT_SECRET=<supabase-jwt-secret>
PRIVATE_SENTRY_DSN=<optional>
PRIVATE_ANTHROPIC_API_KEY=
PRIVATE_OPENROUTER_API_KEY=
PRIVATE_OPENAI_API_KEY=
PRIVATE_XERO_CLIENT_SECRET=
GHCR_REGISTRY_CREDS=<github_username:PAT_with_read_packages>
```

**Platform-managed:** `PUBLIC_REDIS_URL` is injected from the App Platform Valkey binding in `.do/app.yaml` (`${valkey-cache.DATABASE_URL}`). `PRIVATE_DATABASE_URL` is optional; if unset, config builds the URI from `PUBLIC_DB_*` + `PRIVATE_DB_PASSWORD`.

## Required production env vars (Vercel)

```
VITE_BACKEND_URL=https://api.your-domain.com
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=sb_publishable_...
```

---
> Source: [albusOS/sku-ops](https://github.com/albusOS/sku-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
