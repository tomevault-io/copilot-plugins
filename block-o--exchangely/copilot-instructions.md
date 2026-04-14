## exchangely

> This file is the primary operating guide for future agent work in this repository. It describes what Exchangely is, how it works, and how to work in this codebase.

# Exchangely Agent Guide

This file is the primary operating guide for future agent work in this repository. It describes what Exchangely is, how it works, and how to work in this codebase.

## Document Roles

- `AGENTS.md` (this file): Project bible. What Exchangely is, architecture, current state, working rules, validation commands. Start here.
- `ROADMAP.md`: Forward-looking phased roadmap with checkboxes. What we're building next.
- `README.md`: Human-facing overview and quick-start guide.

If these documents disagree:

1. Current code and tests define actual behavior.
2. `AGENTS.md` defines how agents should work in this repo and describes the current system.
3. `ROADMAP.md` defines what to build next.
4. `README.md` is overview material, not the detailed implementation contract.

---

## What Is Exchangely

Exchangely is an event-driven crypto market data platform focused on historical OHLCV coverage for curated crypto/fiat and crypto/stablecoin pairs. It started as a "poor man's CoinGecko" for historical data availability and is evolving into a full-featured exchange rate platform with user accounts, portfolio tracking, price alerts, and a public API.

## Core Stack

- **Backend**: Go single-binary (API + planner + worker roles via `BACKEND_ROLE`)
- **Frontend**: React + Vite dashboard with SSE-driven realtime updates
- **Persistence**: TimescaleDB/PostgreSQL (authoritative source of truth)
- **Transport**: Kafka (task distribution + market events, not coordination authority)
- **Infrastructure**: Docker Compose local topology, GitHub Actions CI

## Architecture Principles

These are non-negotiable unless the user explicitly changes the architecture.

- PostgreSQL/Timescale owns system state. Do not move scheduling truth, sync state, or coordination responsibility into Kafka.
- Planner leadership is lease-based and DB-backed. Keep leadership and scheduling resilient to multiple instances.
- Workers must never mutate the same trading pair concurrently. Pair-level locking is a hard requirement.
- Historical backfill walks backwards from yesterday toward the past. No fixed start date; the system keeps walking back until providers return no data. `BACKEND_PLANNER_BACKFILL_BATCH_PERCENT` caps backfill tasks per planner cycle.
- A daily backfill probe task extends each pair one hour further into the past, discovering new provider data even after regular backfill exhausts known sources.
- `1h` is the preferred canonical historical resolution. Historical fetch granularity must never be coarser than `1d`.
- Month/year rollups, if added, must be derived from canonical stored candles, not provider-native month archives.
- Realtime tasks use a stable ID per pair (`live_ticker:{PAIR}:realtime`). Enqueue logic guarantees at most one pending/running realtime task per pair. Do not add poll-window timestamps to realtime task IDs.
- Prefer SSE for live UI updates when a stream already exists. Do not replace stream-driven flows with aggressive polling.
- The backend is intentionally one binary with role gating. Preserve that unless a deliberate architecture change is requested.
- Keep the project educational and local-first. Do not quietly harden it toward production assumptions without discussing the tradeoff.
- Auth is opt-in via `BACKEND_AUTH_MODE`. When empty, the auth middleware is bypassed and all endpoints are publicly accessible (matching pre-auth behavior). Valid modes are `local` (email/password only), `sso` (Google OAuth only), and `local,sso` (both). The backend validates that required variables are present for the chosen mode at startup and refuses to start if any are missing. When `BACKEND_AUTH_MODE` includes `sso` but `BACKEND_GOOGLE_CLIENT_ID` is empty, startup fails. When it includes `local` but `BACKEND_ADMIN_EMAIL` is empty, startup fails. `BACKEND_JWT_SECRET` is always required when any auth mode is set.
- When running behind a reverse proxy (nginx, Caddy, ALB, etc.), set `BACKEND_TRUSTED_PROXIES` to a comma-separated list of proxy CIDR ranges or IPs so that `X-Forwarded-For` / `X-Real-IP` headers are trusted for resolving the real client IP. This affects rate limiting, audit logging, and fail2ban-style IP blocking.

---

## Current Data Providers

| Provider            | Capabilities          | Notes                                      |
|---------------------|-----------------------|--------------------------------------------|
| Binance             | Historical + Realtime | OHLC + native `/ticker/24hr`               |
| Binance Vision      | Historical            | Bulk CSV archives                          |
| Kraken              | Historical + Realtime | OHLC + native `/Ticker`                    |
| CoinGecko           | Realtime              | `/simple/price` for live quotes            |
| CryptoDataDownload  | Historical            | Hourly/daily CSV fallback                  |

Default quote assets: `EUR` and `USD`.


## What's Built (Current State)

### Backend
- Single Go binary with planner/worker/API role gating
- DB-backed task & lease lifecycle with structured `slog` logging
- Backwards backfill strategy (yesterday → past, no fixed start date)
- Daily backfill probe: one task per pair per day extending coverage into the past
- Realtime dedup: stable per-pair task IDs, at most one live_ticker per pair
- Hourly + daily consolidation pipeline (`FromRaw` → `DailyFromHourly`)
- Advisory pair-level locking for concurrent worker safety
- Multi-layered ticker caching (per-ticker invalidation + time-based global)
- SSE delta streaming for tickers (only changed pairs emitted)
- Task cleanup executor with configurable retention (duration + count)
- Integrity check tasks for cross-source validation
- Gap validation tasks
- News feed ingestion from RSS sources (CoinDesk, Cointelegraph, TheBlock)
- Provider capability system (`CapHistorical`, `CapRealtime`) with registry pre-filtering
- Planner throughput control (backfill batch budget, realtime-first scheduling)
- Worker throughput control (historical-sweep cap per batch)
- Load testing suite integrated into CI
- Google OAuth 2.0 + local admin authentication with JWT sessions
- Role-based access control (admin/user) with Operations panel gating
- Auth middleware with graceful degradation (disabled when `BACKEND_JWT_SECRET` is empty)

### Frontend
- Premium dark-themed dashboard with SSE-driven realtime market updates
- Market Overview with 1h%, 24h%, 7d%, 24h volume, high/low, trend sparklines
- Operations panel with three tabs: Overview (warnings + version), Coverage (coin-grouped), Audit (task history)
- Coverage tab: pairs grouped by base asset in collapsible cards with live feed health, backfill badges, earliest data
- Horizontal scrolling news ticker from RSS feeds
- Vitest testing stack
- Dynamic `__APP_VERSION__` injection

### API Surface
- `GET /api/v1/health`
- `GET /api/v1/assets`
- `GET /api/v1/pairs`
- `GET /api/v1/historical/{pair}` (with `interval`, `start_time`, `end_time`)
- `GET /api/v1/ticker/{pair}`
- `GET /api/v1/tickers`
- `GET /api/v1/tickers/stream` (SSE)
- `GET /api/v1/system/sync-status`
- `GET /api/v1/system/tasks`
- `GET /api/v1/system/tasks/stream` (SSE)
- `GET /api/v1/news`
- `GET /api/v1/news/stream` (SSE)
- `GET /api/v1/auth/google/login` (OAuth redirect)
- `GET /api/v1/auth/google/callback` (OAuth callback)
- `POST /api/v1/auth/local/login` (email/password login)
- `POST /api/v1/auth/refresh` (token refresh via cookie)
- `POST /api/v1/auth/logout` (session invalidation)
- `GET /api/v1/auth/me` (authenticated user profile)
- `POST /api/v1/auth/local/change-password` (password update)
- `GET /api/v1/config` (frontend-facing app configuration: auth state, version)

When changing API behavior, update `router.go:defaultOpenAPIYAML()` if the contract changes materially.

### DevOps
- GitHub Actions CI (Go 1.24 + Node.js 24)
- Docker Compose topology (TimescaleDB, Kafka, backend, frontend)
- Compose-based smoke/e2e tests (`make e2e`)
- Configurable asset catalog via YAML

### Task Types

| Type                  | Description                                                    |
|-----------------------|----------------------------------------------------------------|
| `historical_backfill` | Walks backwards from yesterday; also used for daily probes     |
| `live_ticker`         | Stable ID per pair; at most one pending/running per pair       |
| `integrity_check`     | Cross-source validation for caught-up pairs                    |
| `consolidation`       | Hourly → daily aggregation (future: monthly, yearly)           |
| `task_cleanup`        | Scheduled pruning of completed/failed task logs                |
| `news_fetch`          | RSS feed ingestion from crypto news sources                    |
| `gap_validation`      | Detects missing data windows in synced ranges                  |

---

## Current Runtime Model

Key implementation files:

- `backend/cmd/server/main.go`: backend entrypoint.
- `backend/cmd/migrate/main.go`: migration entrypoint.
- `backend/internal/app/app.go`: application wiring, enabled sources, runtime roles.
- `backend/internal/planner/runtime.go`: leader lease renewal and task scheduling loop.
- `backend/internal/planner/scheduler.go`: task generation logic (backfill, realtime, probes, cleanup).
- `backend/internal/worker/processor.go`: worker claim/execute path.
- `backend/internal/httpapi/router/router.go`: REST and SSE endpoints.
- `backend/internal/storage/postgres/*`: persistence, task state, sync state, leases, locks.
- `frontend/src/components/SystemPanel.tsx`: operations dashboard with three tabs.
- `frontend/src/components/system/CoverageTab.tsx`: coin-grouped coverage view.
- `docker-compose.yml`: local topology and default environment wiring.

## Useful File Map

- `backend/internal/service/*`: application services used by the HTTP layer.
- `backend/internal/ingest/*`: market data providers and registry.
- `backend/internal/messaging/kafka/*`: Kafka producers, consumers, health checks.
- `backend/tests/integration/*`: integration coverage.
- `backend/tests/e2e/*`: compose-backed end-to-end tests.
- `frontend/src/api/*`: frontend API clients.
- `frontend/src/pages/*`: top-level screens.
- `frontend/src/components/*`: UI building blocks.
- `frontend/src/components/system/*`: system operations tab components (OverviewTab, CoverageTab, AuditTab, shared utilities).
- `backend/internal/auth/*`: auth service, JWT, validation, rate limiting.
- `backend/internal/httpapi/middleware/auth.go`: auth middleware.
- `backend/internal/httpapi/handlers/auth.go`: auth HTTP handlers.
- `backend/internal/storage/postgres/user_repository.go`: user persistence.
- `backend/internal/storage/postgres/session_repository.go`: session persistence.
- `frontend/src/app/auth.tsx`: auth context provider.
- `frontend/src/pages/LoginPage.tsx`: login page.
- `frontend/src/pages/SettingsPage.tsx`: settings page.
- `frontend/src/pages/PasswordChangePage.tsx`: password change page.
- `market_dashboard.png`: current dashboard reference image.

---

## Repo-Specific Working Rules

Before starting substantial changes:

- Read this file.
- Check `ROADMAP.md` for current priorities.
- Inspect the actual implementation files you will touch.

When changing backend behavior:

- Respect the planner/worker separation even though they share one binary.
- Keep repository interfaces and domain boundaries coherent; avoid pushing transport or storage concerns deep into domain objects.
- Add or update tests close to the affected package.
- Preserve structured logging with `slog`.
- If you add a provider, wire support through config, source registration, and tests.

When changing frontend behavior:

- Preserve the existing dashboard-first UX and operations visibility.
- Prefer consuming existing SSE streams over introducing new polling loops.
- Keep test coverage in Vitest for non-trivial view logic.
- Maintain the current intentional UI quality; do not downgrade the interface into generic placeholder styling.

When changing configuration:

- Update `backend/internal/config/config.go`.
- Update the Configuration table in `README.md` to reflect the new variable, default, and description.
- Update `docker-compose.yml` if the local stack should expose the setting (add the env var with `${VAR:-default}` syntax).
- Update `.env.example` if the variable is commonly overridden.
- Update example config files when relevant.
- Mention new operational knobs in `AGENTS.md` if they matter to future work.

When changing schema or persistence behavior:

- Add a migration in `backend/migrations`.
- Keep migrations forward/backward paired.
- Validate the affected repositories and any runtime assumptions tied to sync/task state.

## Validation Commands

Use the repo's existing commands instead of inventing one-off workflows.

- `make backend-fmt` / `make backend-fmt-fix`
- `make backend-vet`
- `make backend-lint`
- `make backend-build`
- `make backend-test`
- `make frontend-deps`
- `make frontend-typecheck`
- `make frontend-build`
- `make frontend-test`
- `make check`
- `make test`
- `make e2e`
- `docker compose up --build`
- `docker compose down -v`

E2E notes: `make e2e` uses `scripts/compose-smoke.sh`. The smoke flow validates the live Compose stack plus Kafka topics/consumer groups.

---

## Practical Default For Future Agents

If you are unsure where to put new information:

- Stable repo-specific rules and project state → `AGENTS.md`
- Roadmap items and feature plans → `ROADMAP.md`
- User-facing setup or overview → `README.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/block-o) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
