## rivalclaw

> ArbClaw is a focused arbitrage worker designed to compete against the main OpenClaw trading stack.

# ArbClaw — Specialized Arbitrage Rival Instance

ArbClaw is a focused arbitrage worker designed to compete against the main OpenClaw trading stack.

It is not a general-purpose business OS.

It exists to answer one question:

Can a narrow, fast, mechanical arbitrage system outperform a broader, integrated architecture on real execution-adjusted metrics?

---

## Core Identity

ArbClaw is:
- Mechanical over narrative
- Execution-first over theory-first
- Fast over exhaustive
- Skeptical over optimistic
- Minimal over feature-heavy

ArbClaw is not allowed to be clever at the expense of truth.

---

## Mission

Maximize realized edge capture from arbitrage opportunities.

Not:
- raw PnL
- number of trades
- theoretical edge

Primary objective:

Capture as much real, executable edge as possible with minimal hidden fragility

---

## System Scope

### Included
- Polymarket market ingestion
- Canonical MarketEvent normalization
- Arbitrage detection (single-venue initially)
- Persistence + depth-aware scoring
- Resolution compatibility validation (where applicable)
- Realistic execution simulation
- Paper trading only
- Metrics tracking + export
- CLI + scheduled loop

### Excluded (by design)
- Marketing workflows
- Content generation
- Multi-domain research systems
- Broad agent orchestration
- Unused data feeds (UW, Crucix, etc.)
- Strategy diversification beyond arbitrage
- Live trading

---

## Architecture Philosophy

ArbClaw is an architecture-faithful minimal sibling of MiroFish.

Preserve:
- simulator -> brain -> wallet -> metrics flow
- SQLite-backed state
- graduation gates
- risk/accounting semantics

Remove:
- unused strategies
- unused feeds
- research/intelligence layers
- dashboard complexity

ArbClaw = MiroFish skeleton with only arb organs

---

## Control Flow

Each cycle:
1. Fetch market data
2. Normalize into MarketEvent
3. Run integrity checks (stale/anomaly)
4. Generate candidate opportunities
5. Validate resolution compatibility
6. Evaluate persistence, depth, fees
7. Compute arbitrage score
8. Execute paper trades
9. Check stops / expiry
10. Update wallet state
11. Export metrics

---

## Trading Doctrine

### Fundamental Truth

apparent edge != executable edge != realized edge

ArbClaw only trades when executable edge is positive.

---

A trade is valid ONLY if:
1. Spread exists after fees
2. Spread persists beyond threshold
3. Depth supports intended size
4. Fill probability meets minimum threshold
5. Resolution is compatible (if applicable)
6. No stale or anomalous data
7. Execution assumptions are realistic

If any fail -> REJECT

---

### Fake Arbitrage

ArbClaw assumes most apparent arbitrage is fake.

Common failure modes:
- disappears under size
- disappears under latency
- mismatched resolution wording
- insufficient depth
- stale quotes
- partial fills destroy edge

---

## Execution Model (Paper Only)

Execution simulation must include:
- Slippage (~50 bps baseline)
- Latency penalty (~0.2%)
- Partial fills (80-100%)
- Order book depth constraints

No perfect fills. No optimistic assumptions.

---

## Risk Management

- Max position: 10% of portfolio
- Stop-loss: -20% unrealized
- Take-profit: +50% unrealized
- Auto-close at expiry

No overrides. No exceptions.

---

## Graduation Gates (must match main system)

All must pass:
- Minimum history: >= 14 days
- 7-day ROI > 0%
- Win rate > 55%
- Sharpe ratio > 1.0
- Max drawdown < 25%

ArbClaw does not self-promote to live trading.

---

## Metrics (Truth Layer)

Primary metric:

edge_capture_rate = realized_pnl / expected_pnl

Track:
- Expected PnL
- Realized PnL
- Edge capture rate
- Fill rate
- Slippage
- Missed opportunity PnL
- Opportunity lifetime
- Drawdown
- Capital efficiency

PnL alone is insufficient.

---

## Latency & Performance Tracking

Every cycle must track:
- cycle_started_at_ms
- decision_generated_at_ms
- trade_executed_at_ms

And derived:
- fetch_time_ms
- analyze_time_ms
- execution_time_ms
- total_cycle_time_ms

Goal:

Identify where architecture overhead exists

---

## Data Integrity Rules

Reject any market data that is:
- stale beyond threshold
- internally inconsistent
- structurally impossible (price bounds)
- malformed

ArbClaw must never trade on corrupted input.

---

## Configuration Principles

All optional behavior must be configurable:
- cache vs fresh fetch
- category filtering
- thresholds (edge, persistence, fill)
- cycle cadence

Defaults favor:
- freshness
- safety
- conservatism

---

## Isolation Rules

ArbClaw must be fully isolated:
- separate repo
- separate SQLite DB
- separate logs
- separate positions
- separate signals
- separate scheduler

Shared only:
- public APIs
- comparison contract format
- conceptual strategy rules

No shared runtime state with OpenClaw.

---

## LLM Usage Policy

LLM is restricted.

Allowed only for:
- resolution ambiguity
- edge-case validation
- tie-break decisions
- concise summaries

LLM must NOT:
- generate trade signals
- override scoring rules
- bypass risk filters

---

## Standard Output Contract

ArbClaw exports daily:

```json
{
  "instance_id": "arbclaw",
  "date": "YYYY-MM-DD",
  "expected_pnl": 0.0,
  "realized_pnl": 0.0,
  "edge_capture_rate": 0.0,
  "fill_rate": 0.0,
  "avg_slippage_bps": 0.0,
  "missed_opportunity_pnl": 0.0,
  "avg_opportunity_lifetime_s": 0.0,
  "drawdown_pct": 0.0,
  "capital_efficiency": 0.0,
  "trade_count": 0,
  "false_positive_rate": 0.0,
  "compute_minutes": 0.0,
  "operator_interventions": 0
}
```

This must be compatible with OpenClaw comparison.

---

## Scheduler

Default:
- Run cycle every 10-15 minutes
- Export metrics daily

Avoid unnecessary API pressure.

---

## What ArbClaw Does NOT Optimize For

- Narrative intelligence
- Complex reasoning chains
- Strategy diversity
- Multi-agent coordination
- Market storytelling

It optimizes for:

Reliable, repeatable, execution-realistic arbitrage

---

## Relationship to OpenClaw

ArbClaw is a competitor, not a replacement.

OpenClaw:
- broader
- smarter
- slower
- more context-aware

ArbClaw:
- narrower
- faster
- simpler
- more mechanical

The comparison determines:

Which architecture captures more real edge

---

## Non-Negotiable Rules

- No live trading
- No private key handling
- No bypassing execution realism
- No optimistic fills
- No silent failures
- No trading on ambiguous resolution
- No degradation of metric honesty

---

## Guiding Principle

A strategy that cannot be executed reliably is not a strategy.

A profit that cannot be captured consistently is not real.

---

## Final Mental Model

ArbClaw is:

A disciplined, skeptical arbitrage machine
designed to expose whether simplicity beats integration

If it wins:
-> architecture overhead matters

If it loses:
-> integration + intelligence adds real edge

Either result is valuable.

---

## File Map

```
~/rivalclaw/
├── CLAUDE.md              <- you are here
├── simulator.py           <- orchestrator (fetch -> brain -> wallet -> metrics)
├── trading_brain.py       <- arb-only detection + Kelly sizing + integrity guards
├── paper_wallet.py        <- frozen Mirofish execution sim + latency tracking
├── polymarket_feed.py     <- gamma API + configurable cache/categories
├── graduation.py          <- graduation gates + daily snapshot
├── run.py                 <- CLI entry point (--migrate, --run)
├── daily-update.sh        <- daily report generator + git push
├── daily/                 <- daily performance reports
├── rivalclaw.db           <- SQLite DB (paper_trades, daily_pnl, cycle_metrics, market_data, context)
├── rivalclaw.log          <- cron output log
└── venv/                  <- isolated Python environment
```

## DB Tables

| Table | Purpose |
|-------|---------|
| market_data | Polymarket price snapshots |
| paper_trades | Open/closed positions with granular timestamps |
| daily_pnl | Daily balance/ROI/win rate snapshots |
| cycle_metrics | Per-cycle fetch/analyze/wallet/total timing |
| context | Config state (starting_balance) |

## Cron

- `*/5` — run.py --run (5-minute cycle)
- `23:45` — daily-update.sh (daily report + git push)

## Repos

- This instance: https://github.com/nayslayer143/rivalclaw
- Lean baseline: https://github.com/nayslayer143/arbclaw
- Main system: https://github.com/nayslayer143/clawmpson-logs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nayslayer143)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nayslayer143)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
