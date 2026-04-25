## crawlsight

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Scraper Insight is a self-hosted observability platform for high-scale web crawlers. It provides real-time dashboards, log ingestion, analytics, alerting, and an MCP server for AI tools.

- **Backend**: Go 1.24+ with Echo framework, MongoDB database, NATS message queue
- **Frontend**: Next.js 15 (static export) with React 19, Tailwind CSS 4, and Chart.js
- **Auth**: Username/password with bcrypt + JWT tokens
- **Real-time**: WebSocket with NATS bridge for live dashboard updates
- **Alerts**: In-process rule engine evaluating metrics via sliding windows
- **MCP**: Stdio MCP server for AI tool integration
- **Deployment**: Docker Compose with MongoDB, NATS, and app container

## Build & Run Commands

```bash
# Build everything
make all

# Build only
make build-ui    # Next.js → ui/out
make build-go    # Go binaries → bin/server, bin/mcp-server

# Run locally (requires MongoDB + NATS running)
make run

# Frontend dev server (separate from Go backend)
cd ui && npm run dev    # next dev on port 3000

# Frontend lint
cd ui && npm run lint

# Go tests
go test ./...

# Docker (full stack)
docker compose up

# Docker build only
make docker-build
```

## Local Development Setup

Start dependencies:
```bash
docker compose up mongo nats
```

Configure `.env.local`:
```env
PORT=3000
MONGO_URI=mongodb://localhost:27017
MONGO_DATABASE=scraper_insight
NATS_URL=nats://localhost:4222
JWT_SECRET=any-random-string
ADMIN_USERNAME=admin
ADMIN_PASSWORD=changeme
STATIC_DIR=./ui/dist/ui/browser
```

## Architecture

### Request Flow

**Crawler ingestion**: `POST /api/v1/report` → machine auth middleware (`X-App-ID` + `X-App-Secret` headers) → publish JSON to NATS `scraper.logs.ingest` → ingestion writer batches into MongoDB (100 entries or 1s flush).

**User dashboard**: Angular SPA served as static files → JWT auth interceptor attaches token → `auth.guard.ts` checks `/api/v1/me` before every protected route.

**Real-time**: WebSocket at `/api/v1/ws?token=JWT` → NATS subscribers bridge log entries, alerts, and periodic stats updates to connected clients.

**Alerts**: Background goroutine evaluates alert rules every 5s against sliding windows of log entries. Fired alerts publish to NATS `scraper.alerts.triggered` and broadcast via WebSocket.

### Key Packages

| Path | Responsibility |
|------|---------------|
| `cmd/main.go` | Echo server setup, NATS/MongoDB/WebSocket wiring, route registration |
| `cmd/mcp/main.go` | Stdio MCP server binary with query tools |
| `pkg/config/config.go` | Env var loading into `Config` struct |
| `pkg/db/interface.go` | `StorageInterface` — contract for all storage operations |
| `pkg/db/models.go` | All data types: `LogEntry`, `App`, `User`, `AlertRule`, `Alert`, stat types |
| `pkg/db/mongo.go` | MongoDB implementation of `StorageInterface` |
| `pkg/nats/client.go` | NATS connection wrapper with reconnect handling |
| `pkg/ingestion/writer.go` | NATS → MongoDB batch writer |
| `pkg/websocket/hub.go` | WebSocket client hub with broadcast |
| `pkg/websocket/client.go` | WebSocket client read/write pumps |
| `pkg/alerts/evaluator.go` | Sliding window alert rule evaluation engine |
| `internal/auth/jwt.go` | JWT token generation and validation |
| `internal/auth/auth_handler.go` | Login, register, me, logout endpoints |
| `internal/middleware/jwt_auth.go` | JWT validation middleware (Bearer header or cookie) |
| `internal/middleware/machine_auth.go` | Crawler API key validation |
| `internal/handler/stats.go` | Stats & app management endpoints |
| `internal/handler/report.go` | Log ingestion handler (legacy direct-to-storage) |
| `internal/handler/alerts.go` | Alert rule CRUD and alert management |
| `internal/handler/websocket.go` | WebSocket upgrade handler |

### Database Schema (MongoDB)

| Collection | Type | Key fields |
|-----------|------|------------|
| `crawling_logs` | Regular (TTL 30d) | `app_id`, `domain`, `url`, `status_code`, `method`, `error_msg`, `latency_ms`, `created_at` |
| `apps` | Regular | `_id` (UUID), `secret`, `name`, `created_by`, `created_at` |
| `users` | Regular | `_id` (username), `password_hash`, `display_name` |
| `alert_rules` | Regular | `name`, `metric`, `operator`, `threshold`, `window_seconds`, `app_id`, `enabled` |
| `alerts` | Regular | `rule_id`, `actual_value`, `message`, `resolved`, `triggered_at` |

### NATS Subjects

- `scraper.logs.ingest` — raw log entries from report handler
- `scraper.alerts.triggered` — fired alerts

### Auth Details

- **JWT Auth**: Username/password login returns JWT token (7-day expiry). Token sent via `Authorization: Bearer` header or `token` cookie. Admin user seeded from `ADMIN_USERNAME`/`ADMIN_PASSWORD` env vars on startup.
- **Machine auth**: Crawler registers an app via `POST /api/v1/apps`, gets an `id` + `secret`. All subsequent `/api/v1/report` calls must include `X-App-ID` and `X-App-Secret` headers.

### MCP Tools

| Tool | Description | Parameters |
|------|-------------|------------|
| `query_stats` | Global KPIs | `days?`, `app_id?` |
| `list_apps` | All scraper apps with stats | — |
| `get_app_details` | Per-app stats + domains | `app_id` |
| `list_errors` | Recent errors | `app_id?`, `limit?`, `domain?` |
| `get_scraper_status` | Is scraper active? | `app_id` |
| `list_domains` | Domain aggregation | `limit?`, `app_id?` |
| `list_alerts` | Recent alerts | `resolved?`, `limit?` |
| `manage_app` | Create/delete app | `action`, `name?`, `app_id?` |

### Alert Metrics & Operators

**Metrics:** `error_rate`, `latency_avg`, `error_count`, `request_count`
**Operators:** `gt`, `lt`, `gte`, `lte`, `eq`

## CI/CD

GitHub Actions (`.github/workflows/docker-publish.yml`) builds and pushes to GHCR:
- `ghcr.io/.../scraper-insight:latest` on pushes to `main`
- `ghcr.io/.../scraper-insight:<tag>` on git tags

## Frontend Structure

```
ui/src/
  app/
    layout.tsx               # Root layout with fonts, metadata
    client-layout.tsx        # Client wrapper with AppShell
    app-shell.tsx            # Auth guard, sidebar, breadcrumbs, WebSocket
    globals.css              # Tailwind imports, CSS custom properties, design tokens
    page.tsx                 # Dashboard (KPIs, charts, live feed)
    login/page.tsx           # Split-layout login form
    apps/page.tsx            # App list + creation/credentials modals
    apps/detail/page.tsx     # Per-app stats, bar chart, log table (via ?id=)
    alerts/page.tsx          # Alert rules CRUD + alert history (tabbed)
  components/
    sidebar.tsx              # Sidebar navigation with theme toggle
    breadcrumb-bar.tsx       # Breadcrumb + WebSocket connection indicator
    alert-toast.tsx          # Alert toast notifications
    shared/
      badge.tsx              # Status badges (success/danger/warning/info)
      kpi-card.tsx           # KPI metric card with icon + trend
      empty-state.tsx        # Empty state with icon + CTA
      modal.tsx              # Modal dialog with overlay
      toggle.tsx             # Toggle switch
  hooks/
    use-auth.ts              # Auth context + session management
    use-websocket.ts         # WebSocket hook with auto-reconnect
    use-theme.ts             # Dark/light mode toggle
  lib/
    api.ts                   # API client with JWT auth + 401 handling
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/autofacts) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
