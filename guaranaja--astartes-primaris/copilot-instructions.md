## astartes-primaris

> Cloud-native multi-asset algorithmic trading platform — the command and control layer for the Astartes ecosystem.

# ASTARTES PRIMARIS — Claude Context

## What This Is
Cloud-native multi-asset algorithmic trading platform — the command and control layer for the Astartes ecosystem.
All trading engines (futures, equities) report to Primaris. Primaris orchestrates, monitors, and manages.

## Ecosystem Architecture

| Repo | Codename | Role |
|------|----------|------|
| `astartes-primaris` | **Primaris** | Command & control platform |
| `astartes-futures` | **Fortress Primus** | Futures trading engine (ES) |
| `astartes-futures-client` | **Fortress Primus Client** | Futures client distribution |
| `astartes-equities` | **Fortress Secundus** | Equities + options engine |
| `astartes-equities-client` | **Fortress Secundus Client** | Equities client distribution |

## Strategy Names (Space Marine Chapter inspired)

| Name | Chapter | Asset | Strategy |
|------|---------|-------|----------|
| Eversor | Assassin Temple | Futures | RTH intraday |
| Fenris | Space Wolves | Futures | Globex London overnight |
| Stormchaser | White Scars | Equities | Momentum/breakout |
| Phalanx | Imperial Fists | Equities | Wheel/options income |
| Corax | Raven Guard | Equities | Mean reversion |
| Medusa | Iron Hands | Equities | Systematic |
| Nocturne | Salamanders | Equities | Value/swing |

## Services

| Service | Port | Language | Purpose |
|---------|------|----------|---------|
| Primarch | 8401 | Go | Control plane, orchestrator, REST API |
| Logis | 8601 | Go | Order execution, position tracking |
| Librarium | 5432 | TimescaleDB | Market data, trades, positions, all persistent state |
| Auspex | 8501 | Python | Market data collection (IBKR, TastyTrade) |
| Vox | 4222 | NATS | Event bus (inter-service messaging) |
| Aurum | 3000 | Next.js | Web dashboard |
| Forge | 8701 | Python | Backtest & optimization |
| Registry | 8092 | Go | Strategy marketplace, billing |
| Codex | — | Go | Configuration service |
| Apothecary | 9090/3001 | Prometheus/Grafana | Monitoring |
| Iron Halo | — | Vault | Secrets management |
| CFO Engine | 8080 | Firefly III | Personal finance tracking |

## Key Files

```
schemas/protobuf/imperium.proto        — Core data models (Fortress, Marine, Signal, Order, Position)
schemas/protobuf/services.proto        — gRPC service contracts (Primarch, Logis, Forge, Codex)
services/librarium/migrations/         — Database schema (TimescaleDB)
services/primarch/internal/            — Control plane implementation
services/primarch/internal/cfo/        — Financial hub (Firefly III + Monarch Money clients)
services/primarch/internal/api/        — REST API endpoints
services/aurum/                        — Next.js dashboard
sdk/marine-sdk/                        — Python SDK for strategies
ecosystem/                             — Shared patterns, Model Zoo, MCP Gallery
ecosystem/shared/sync_contract.py      — Typed engine→Primaris sync dataclasses
ecosystem/model-zoo/INDEX.md           — ONNX model registry
ecosystem/mcp-gallery/GALLERY.md       — MCP server catalog
docker-compose.yml                     — Local dev stack
```

## Ecosystem Improvement Protocol

When you make an improvement to a shared pattern (regime detection, confidence
scoring, risk management, indicator calculation, trailing stops, etc.):

1. **Tag it**: Add comment `# ECOSYSTEM: <pattern-name>` near the change
2. **Log it**: Append to `ecosystem/CHANGELOG.md` with:
   - What changed and why
   - Which repo originated the change
   - Which other repos should adopt it
3. **Propagate**: Open issues in affected repos describing the improvement

Shared patterns live canonically in `ecosystem/shared/`.
Asset-specific repos may have adapted versions, but core logic should stay in sync.

## Hard Rules

1. **Never return raw dicts at API boundaries** — always typed dataclasses or protobuf
2. **Schema migrations are backward-compatible** — ALTER ADD only, never ALTER DROP
3. **New proto fields use high field numbers** (10+) to avoid conflicts
4. **astartes-futures changes are additive only** while combine is live
5. **Test fill price propagation end-to-end** after any execution path change

## Financial Hub (Council)

- Firefly III = finance engine (accounts, budgets, bills, goals)
- Monarch Money = family finance (read-only — balances, cash flow)
- Council CFO = orchestrator unifying both + trading data
- Coverage bubble: tradingIncome / totalExpenses = % of bills covered
- Payout allocations: ledger entries, no programmatic transfers

---
> Source: [guaranaja/astartes-primaris](https://github.com/guaranaja/astartes-primaris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
