## gimble

> Gimble — monorepo for robotics debugging and observability


# Gimble

Monorepo for robotics debugging and observability. Packages follow the `gimble-*` pattern (e.g., gimble-ros, future gimble-sim, gimble-isaac).

## Purpose

Collect, correlate, and reason over live system state from robotics systems:
- System topology and connectivity snapshots
- Log output and diagnostic health checks
- Trace correlation across processes and time

The collected data forms a **dynamic system context graph** that an AI agent queries to surface issues, correlate anomalies across subsystems, and guide the user toward the root cause.

## Packages

| Package | Description |
|---------|-------------|
| gimble-ros | ROS2 context collection, CLI (`gim`), telemetry HTTP server |
| (future) | Additional platforms (simulation, Isaac, etc.) |

## Config Layout

- Base: `~/.config/gimble/`
- Package-scoped: `~/.config/gimble/<package>/` (e.g., `ros/` for gimble-ros)
- See `.codex/rules/platform.mdc` for full config precedence

## Repos

| Repo | Path | Role |
|------|------|------|
| gimble | This repo | Monorepo — gimble-ros and future gimble-* packages |
| Gimble-dev | Sibling | Infra, cloud chat, profiles |

## AI Agent Integration

The agent reads artifacts from `~/.config/gimble/<package>/` or via the HTTP API:
- `POST /analyze` — full system snapshot + summary
- `GET /graph` — live topology graph
- `GET /diagnostics` — health and metrics

The AI agent's API key is injected via environment variable — never hardcoded.

## Codex Agent

A separate **Codex agent** handles release, packaging, and git tagging for this repo.
Rules live in `.codex/rules/`. Do not touch release tags or changelogs manually.
When making structural changes (new collectors, new API endpoints), update `.codex/rules/`
if the change affects build targets, artifact naming, or validation steps.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RN0H) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
