## claude-code-monitoring

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Monitoring is a Docker-based observability stack for tracking Claude Code CLI usage metrics and events. The stack receives OpenTelemetry data from Claude Code, stores metrics in Prometheus and events in Loki, and visualizes them in Grafana.

```
Claude Code CLI ──OTLP──→ OTel Collector (:4317) ──→ Prometheus (:9090) ──→ Grafana (:3030)
                                │                                              ↑
                                └── logs ──→ Loki (:3100) ─────────────────────┘
```

**Author:** Rommel C. Porras (https://rommelporras.com)
**Repository:** https://github.com/rommelporras/claude-code-monitoring
**Official Docs:** https://code.claude.com/docs/en/monitoring-usage

## Key Commands

```bash
# Stack management
./start.sh              # Start all containers
./start.sh status       # Check health (containers + HTTP endpoints)
./start.sh stop         # Stop all containers
docker compose restart claude-grafana  # Reload dashboard changes

# Verify metrics flowing
curl -s "http://localhost:9090/api/v1/query?query=claude_code_cost_usage_USD_total" | jq
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job, health}'
```

## Critical Architecture Knowledge

### Per-Session Counter Lifecycle

**CRITICAL:** Metrics are **per-session counters**. When a Claude Code session ends:
- The time series becomes **stale** (no new data points)
- Instant queries (`query`) exclude stale series
- Range queries (`query_range`) still include historical data

**Always use `increase()` or `rate()` for aggregations — never raw `sum()`:**
```promql
# WRONG - excludes ended sessions
sum(claude_code_cost_usage_USD_total)

# CORRECT - includes all sessions in time range
sum(increase(claude_code_cost_usage_USD_total[$__range]))
```

### Datasource UIDs

Both datasources use pinned UIDs — use these in all dashboard panel definitions:
- **Prometheus:** `"uid": "prometheus"`
- **Loki:** `"uid": "loki"`

### LogQL with OTLP Native Ingestion

The OTel Collector uses `otlphttp/loki` exporter which sends data via OTLP native ingestion. Event attributes become **structured metadata labels** — no `| json` parsing needed:

```logql
# Correct — structured metadata filter
{service_name="claude-code"} | event_name="api_request"

# WRONG — json parsing is unnecessary with OTLP native ingestion
{service_name="claude-code"} | json | event_name="api_request"
```

### Dashboard Provisioning

- **`dashboard.yml`** has `editable: false` — Grafana ignores UI edits on restart, JSON file is source of truth
- **`claude-code-dashboard.json`** has `"editable": true` — allows users to experiment in the UI
- These are intentionally different: provisioning controls reload behavior, dashboard flag controls UI

To make permanent dashboard changes:
1. Edit `claude-code-dashboard.json` directly
2. `docker compose restart claude-grafana`

### Dashboard Structure

- **8 sections** (collapsible rows): Overview, Productivity, Sessions & Activity, Trends, Cost Analysis, Performance (Events), Token & Efficiency, Insights
- **33 content panels** across all sections
- **24-unit wide grid** layout: `gridPos: {x, y, w, h}`
- Table panels require `format: "table"` and `instant: true`
- Table `sortBy` must reference the **renamed column name**, not the original field

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Tables over bar gauges | Bar gauges don't support value-based sorting for multi-series data |
| `user_email` over `user_id` | Readable email addresses vs cryptic hashes in queries |
| Pinned Docker image versions | `latest` can break the stack silently |
| Billing cycle variable | More useful than rolling 30-day window for subscription tracking |

## Deployment Modes

This project supports two deployment modes:

| Mode | Config | Prometheus | Loki | Grafana |
|------|--------|-----------|------|---------|
| **Docker Compose** | `docker-compose.yml` | Bundled container | Bundled container | Bundled container |
| **Kubernetes** | See `docs/kubernetes.md` | Existing kube-prometheus-stack | Existing Loki | Existing Grafana (ConfigMap sidecar) |

## Rules

- **Pin Docker image versions** — never use `latest` in docker-compose.yml
- **Security review before every commit** — scan for leaked passwords, tokens, API keys
- **Dashboard JSON is source of truth** — never edit dashboards through Grafana UI for permanent changes
- **Use `increase()` or `rate()`** for all metric aggregations — never raw `sum()`

## Version History

- v2.0.0 — Loki events, 33-panel dashboard (8 sections), billing cycle tracking, K8s guide
- v1.1.1 — Fix container auto-restart on Windows Docker Desktop
- v1.1.0 — Dashboard v2 with improved layout and productivity metrics
- v1.0.0 — Initial release

---
> Source: [rommelporras/claude-code-monitoring](https://github.com/rommelporras/claude-code-monitoring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
