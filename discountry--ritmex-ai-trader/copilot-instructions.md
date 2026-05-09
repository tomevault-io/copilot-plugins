## ritmex-ai-trader

> This project targets an AI-driven trading assistant. The document sketches a modular agent architecture so each responsibility can evolve independently while crossing data through well-defined interfaces.

# Agent Roster

## Overview
This project targets an AI-driven trading assistant. The document sketches a modular agent architecture so each responsibility can evolve independently while crossing data through well-defined interfaces.

## Shared Contracts
- **Message Bus:** JSON messages on an internal queue (e.g. Redis Streams, Bun `EventTarget`). Each payload carries `timestamp`, `source`, `traceId`.
- **State Store:** SQLite or DuckDB snapshot for historical features, streamed updates cached in Redis.
- **Model Registry:** Local directory or OCI bucket with versioned model artefacts referenced by semantic tags (`model@vYYMMDD`).

## Agents

### 1. Market Data Ingestion Agent
- **Purpose:** Subscribe to exchanges/brokers, normalize tick/ohlcv feeds.
- **Inputs:** REST/WebSocket streams, static instrument metadata.
- **Outputs:** `market.raw` messages; updates `market_data` tables.
- **Key Ops:** Rate-limit enforcement, instrument calendar alignment, missing-bar backfill.

### 2. Feature Engineering Agent
- **Purpose:** Transform raw market events into model-ready features.
- **Inputs:** `market.raw`, indicator specs (`docs/indicators.ts`), macro calendars.
- **Outputs:** `features.candles`, `features.micro` vectors with metadata for the Signal agent.
- **Key Ops:** Rolling window computation, anomaly filtering, unit tests on indicator drift.

### 3. Signal Generation Agent
- **Purpose:** Run ML/statistical models to emit trade signals.
- **Inputs:** Feature batches, model version references, risk parameters.
- **Outputs:** `signal.long`, `signal.short`, `signal.flat` messages with confidence scores (`0..1`).
- **Key Ops:** Ensemble voting, regime classification, shadow evaluation vs. benchmark.

### 4. Portfolio & Risk Agent
- **Purpose:** Convert signals into position targets under risk constraints.
- **Inputs:** Signal stream, portfolio state, risk settings (max leverage, VAR limits).
- **Outputs:** `portfolio.target` events (instrument, size, ttl), risk alerts.
- **Key Ops:** Kelly/leverage scaling, drawdown monitoring, breach escalation to Supervisor.

### 5. Execution Agent
- **Purpose:** Stage and route orders to venues with minimal slippage.
- **Inputs:** Portfolio targets, market microstructure data, venue configs.
- **Outputs:** Order tickets, fill confirmations (`execution.fill`), venue error reports.
- **Key Ops:** TWAP/VWAP scheduling, smart order routing, kill-switch on connectivity loss.

### 6. Supervisor Agent
- **Purpose:** Orchestrate agents, enforce SLAs, manage health checks.
- **Inputs:** Heartbeat pings, log metrics, anomaly alerts.
- **Outputs:** Recovery commands, incident reports, shift handoff notes.
- **Key Ops:** Circuit breaker management, chaos tests, audit trail consolidation.

### 7. Compliance & Reporting Agent
- **Purpose:** Maintain audit-ready logs and regulatory reports.
- **Inputs:** Execution logs, risk overrides, configuration diffs.
- **Outputs:** Daily reports, trade blotters, retention-ready archives.
- **Key Ops:** P&L reconciliation, regulatory schema validation, immutable storage write.

## Communication Diagram
```
Market Data → Feature → Signal → Portfolio/Risk → Execution
                                     ↓
                              Compliance/Reporting
        ↑                               ↑
     Supervisor ← health/alerts ← all agents
```

## Deployment Notes
- Start with Supervisor, then Market Data, Feature, Signal, Portfolio, Execution, Compliance.
- Prefer containerized deployment; agents expose HTTP control endpoints for readiness/liveness.
- Define `.env` per agent mirroring shared secrets through Vault or SSM; never bake credentials into images.

## Next Steps
1. Decide target venues/brokers so the Market Data and Execution agents can load specific adapters.
2. Implement message schemas (`docs/signal.md` as initial reference) and publish via a shared package.
3. Align feature computation with Python prototype in `docs/demo.py`, then port to TypeScript for runtime parity.

---
> Source: [discountry/ritmex-ai-trader](https://github.com/discountry/ritmex-ai-trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
