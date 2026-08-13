## opencodego-pool

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

OpenCodeGoPool is an account pool manager for [opencode.ai](https://opencode.ai) accounts. It periodically scrapes quota and usage data from multiple opencode.ai accounts via their browser cookies, and syncs all active API keys to an external CPA (Claude Proxy Aggregator) endpoint. The entire app ships as a single Go binary with an embedded React frontend.

## Commands

### Backend (Go)

```bash
# Run the server (reads config.yaml from cwd)
go run ./cmd/server

# Run all tests
go test ./...

# Run tests for a specific package
go test ./internal/cpa/...

# Vet
go vet ./...
```

### Frontend (web/)

```bash
cd web
npm install        # or npm ci
npm run dev        # Vite dev server (proxies /api to :8080)
npm run build      # TypeScript check + Vite build
npm run lint       # OxLint
```

### Full production build (Windows)

```bat
build.bat
```

This script: `npm ci` → `npm run build` → copies `web/dist` to `internal/frontend/dist` → `go test ./...` → `go vet ./...` → cross-compiles to `build/opencode-pool-linux-amd64` with `CGO_ENABLED=0`.

## Architecture

### Data flow

```
Scheduler (every 5 min)
  └─► Scraper.FetchQuota(cookie, workspaceID)
        └─► opencode.ai/workspace/{id}/go  → parse $R JS blobs → QuotaSnapshot
  └─► Scraper.FetchUsage(cookie, workspaceID)
        └─► opencode.ai/workspace/{id}/usage → parse $R JS blobs → []UsageRecord
  └─► Scraper.FetchUsageHistoryMonth(...)
        └─► Server Function API → []UsageDailyHistory
  └─► If limit exceeded changed → cpa.Syncer.Sync()

CPA Syncer
  GET external endpoint → []CPAProvider
  Merge own provider entry (active accounts' API keys only)
  PUT merged list back
```

### Key packages

| Package | Role |
|---|---|
| `internal/api` | chi HTTP router + handlers. Session auth via in-memory `SessionStore`. All routes under `/api/v1`, protected by `authMiddleware`. |
| `internal/scraper` | HTTP client that fetches opencode.ai pages. Parser (`parser.go`) extracts embedded React Server Component data (`$R[n]` assignments) from inline `<script>` tags, normalizes the JS to valid JSON, then deserializes. |
| `internal/scheduler` | Ticker-based background loop. Runs `ScrapeAll` immediately on start, then every `scraper.interval`. Calls `cpa.Sync` automatically when an account's `limit_exceeded` flag changes. |
| `internal/cpa` | Syncer that reads CPA settings + active accounts from the store, fetches the remote provider list, merges the own-provider entry, and PUTs the result. Merge is non-destructive: other providers in the remote list pass through unchanged. |
| `internal/store` | SQLite via `modernc.org/sqlite` (pure Go, no CGO). Schema is applied idempotently via `migrate.go`; columns added after the initial release use `ALTER TABLE ... ADD COLUMN` with duplicate-column error suppression. `Store` is an interface — inject it in tests using a real in-memory SQLite (`t.TempDir()`). |
| `internal/model` | Plain structs for DB rows and API payloads. `DefaultCPAModels` lists the model names synced to CPA. |
| `internal/frontend` | Contains only `embed.go`, which embeds `dist/` at compile time. The `dist/` directory must exist before `go build`. |
| `internal/config` | YAML config loader with hardcoded defaults (`:8080`, `./data/pool.db`, 5 min interval). Missing config file is silently accepted. |

### Frontend structure (`web/src/`)

- `api/client.ts` — typed `fetch` wrapper; auto-redirects to `/login` on 401
- `api/types.ts` — TypeScript mirrors of Go model structs
- `components/` — organized by feature: `accounts/`, `dashboard/`, `usage/`, `settings/`, `auth/`, `layout/`, `common/`, `ui/` (shadcn primitives)
- `providers/` — `AppProviders` wraps the app in React Query + Router + Theme + Toaster
- No global state management beyond React Query's cache

### Database schema (SQLite)

- `accounts` — email, cookie, workspace_id, api_key, status, quota limit thresholds, `limit_exceeded` flag
- `quota_snapshots` — rolling/weekly/monthly percent + status + reset-sec per account per scrape
- `usage_records` — per-request records (model, tokens, cost in micro-units) keyed on `(id, account_id)`
- `usage_daily_history` — aggregated daily cost per `(account_id, date, model, key_id)`
- `cpa_sync_log` — append-only log of sync attempts
- `settings` — key/value store used for CPA settings and per-account bootstrap flags

### Cost units

Token costs are stored as integers in **micro-units of USD** (divide by `100_000_000` to get dollars).

## Configuration

Copy `config.example.yaml` to `config.yaml`. Required fields:

- `server.password` — dashboard login password
- `cpa.endpoint` — the CPA management API URL
- `cpa.bearer_token` — auth token for the CPA API

The scraper uses session cookies from opencode.ai. Accounts with cookies that return 401/302 are automatically marked `error`.

## Testing approach

Tests use real SQLite databases created in `t.TempDir()` — no mocks for the store layer. HTTP dependencies (CPA endpoint) are tested with `httptest.NewServer`. See `internal/cpa/sync_test.go` for the pattern.

---
> Source: [xnqycs/OpenCodeGo_Pool](https://github.com/xnqycs/OpenCodeGo_Pool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
