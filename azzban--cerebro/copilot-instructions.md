## project-overview

> Core project context, architecture, and tech-stack overview for Cerebro


# Cerebro — Project Overview

**Cerebro** is a Go CLI automated trading system for Binance (spot + futures) with paper-trade-first safety, a multi-agent LLM layer, and a terminal TUI.

## Tech Stack

| Concern | Choice |
|---|---|
| Language | Go 1.25 (module: `github.com/azhar/cerebro`) |
| CLI | `cobra` |
| TUI | `bubbletea` + `lipgloss` |
| Exchange | `go-binance/v2` |
| DB | `pgx/v5` (PostgreSQL) |
| Cache | `go-redis/v9` |
| Config | `yaml.v3` + `godotenv` |
| LLM | `go-openai` + custom Anthropic/Gemini HTTP clients |
| ChatOps | `telegram-bot-api/v5`, `discordgo` |
| Numerics | `shopspring/decimal` — **never `float64` for money** |
| IDs | `google/uuid` |
| Concurrency | `golang.org/x/sync/errgroup` |

## Architecture

Strict **hexagonal (ports-and-adapters)**:

```
cmd/cerebro/main.go → internal/cli → internal/app (composition root)
   ├─ internal/domain     (pure types, no deps)
   ├─ internal/port       (interfaces only)
   ├─ internal/adapter    (implementations: binance, postgres, redis, llm, bots)
   ├─ internal/config     (load + validate)
   └─ internal/app        (wiring + lifecycle via errgroup)
```

## Safety Invariants

- Paper mode is **default and mandatory** until live path is fully implemented.
- Triple agreement required to go live: `ENVIRONMENT=production` in secrets, `environment: production` in `app.yaml`, and `--live` flag.
- Kill-switch (`engine.kill_switch: true`) halts all execution immediately.
- **Never bypass the risk gate** (`internal/risk`) in execution paths.

## Key Directories

- `internal/domain/` — enums, value types, no outward imports
- `internal/port/` — Go interfaces (ports)
- `internal/adapter/` — external system implementations
- `internal/app/runtime.go` — composition root / goroutine wiring
- `configs/` — `app.yaml`, `markets.yaml`, `strategies.yaml`, `secrets.env`
- `scripts/migrations/` — Goose SQL migrations
- `deploy/` — `Dockerfile`, `docker-compose.yaml`

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
