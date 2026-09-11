## hyperliquid-market-maker-bot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Go bot for market-making on Hyperliquid perpetual markets — both HIP-3 dex assets (e.g. `xyz:CL`) and standard perps (e.g. `BTC`, `ETH`). Always quotes both sides, uses short-window VWAP as the primary fair-value anchor, and earns bid-ask spread with mean-reversion on VWAP wicks.

**The shipped `config.example.yaml` targets a single asset (`xyz:CL`).** The bot's code still supports multi-asset / spread-pair mode (mixed HIP-3 + standard perps in one process, optional z-score cross-leg spread engine), and every relevant subsystem is per-asset aware (`Config.ForAsset(coin)`, per-asset optimizer/risk/MM overrides). To run multiple assets, add them to `strategy.assets` and — if they are correlated — leave `spread_engine_enabled` at its default (auto-on with 2+ assets) or set it explicitly.

Runtime config lives in `config.yaml` (copied from `config.example.yaml`, never committed). High-level features: adaptive notional, z-score calibration, trade-flow imbalance, post-fill decay, regime detection, markout tracking, per-side toxicity scoring with idle decay, inventory half-life controller, entry-side size reduction, dynamic refresh intervals, volume-triggered burst ladders, a unified utility-driven optimizer (4D state, policy table, counterfactual rollback), and a rate-limit-aware API budget allocator.

### Canonical Docs

- `docs/ALGORITHM.md` — full algorithm spec; **update this whenever the core algorithm changes**
- `docs/CONFIG.md` — per-field configuration reference
- `docs/GETTING_STARTED.md` — step-by-step operator walkthrough
- `config.example.yaml` — source of truth for the config schema (every field is annotated inline)
- `README.md` — user-facing overview and quick-start
- `AGENTS.md` is a symlink to `CLAUDE.md` — editing one updates the other

## Rules

- Any changes to the core algorithm should be documented by updating `docs/ALGORITHM.md` so it stays current
- All configs must print on startup so they land in the logs — keep this in mind when adding or removing configs (see `main.go` config-dump section)
- If you change any log fields or messages, update `cmd/monitor/parser.go` accordingly (or verify the monitor still parses correctly) — the TUI is a pure log consumer and will silently drop unknown fields

## Commands

Build artifacts land in `bin/` (gitignored).

```bash
make build                # go build -o bin/bot ./cmd/bot
make run                  # build + run bin/bot with config.yaml
make run-with-logs        # build + run bin/bot, tee to logs/logs.txt
make monitor              # build monitor TUI into bin/monitor
make run-monitor          # build + run bin/monitor (auto-discovers latest log in logs/)
make test                 # go test ./... -count=1
make test-race            # go test ./... -count=1 -race
make test-v               # go test ./... -count=1 -v
make vet                  # go vet ./...
make format               # go fmt ./...
make lint                 # go vet + staticcheck (if installed)
make validate             # build + validate config.example.yaml (bin/bot -validate; credential checks skipped so this works offline)
make check                # vet + validate + test-race
make clean                # rm -rf bin/
go run ./cmd/bot/ -config config.yaml     # run directly without building

# Run a single test
go test ./internal/spread/ -run TestEngineLongSpread -v

# Diagnostic: dump raw clearinghouse state (perps, dex, spot)
go run ./cmd/diag/

# Diagnostic: dump recent trades
go run ./cmd/diag-trades/

# Diagnostic: stream L2Book depth
go run ./cmd/diag-book/

# Monitor: TUI dashboard for real-time log monitoring
go run ./cmd/monitor/                    # auto-discover latest log in logs/
go run ./cmd/monitor/ logs/logs-cl-v11.txt  # open specific log file
```

## Architecture

Single binary, event-driven pipeline. All components communicate via typed Go channels with non-blocking sends (buffered channels, size 100). A single `context.Context` governs graceful shutdown via SIGINT/SIGTERM.

```
gRPC L2Book ──> fan-out ──> Spread Engine ──> continuous z-score ──> Quote Manager ──> Executor
                  │                                                       ^               │
                  ├──> Quote Manager (mid prices, book depth)             │               v
                  ├──> Live Executor (book snapshots for slippage close)  │         PositionUpdate
                  └──> Paper Executor (fill sim)                    RiskAction            │
                                                                         │               │
Trade gRPC    ──> Short VWAP ──> Quote Manager                    Risk Manager <─────────┘
              ├──> Long VWAP ──> Quote Manager
              └──> Trade-Flow Imbalance ──> Quote Manager

UserFills WS ──> Live Executor (fill reconciliation)
                  └──> Fill Callback ──> FillDecay, FillRateTracker, MarkoutTracker,
                                         SideToxicityTracker
```

### API Routing

- **QuickNode** -- gRPC streams (L2Book), WebSocket streams (UserFills, Trades), Info queries
- **hyperliquidapi.com** -- all trading operations. The Go SDK routes these automatically via `send.hyperliquidapi.com`
- SDK initialized with QuickNode endpoint; trading calls route to hyperliquidapi.com internally

### Key Data Flow

1. **gRPC feed** (`feed/grpc.go`) streams L2Book for configured assets, fans out `BookUpdate` to spread engine (if 2+ assets), quote manager, live executor (live mode), or paper executor (paper mode)
2. **Trade feed** (`feed/trades_grpc.go`) streams public trades via gRPC, fanning out to per-asset short VWAP, long VWAP, and trade-flow imbalance calculators via a goroutine in `main.go`.
3. **WebSocket fill feed** (`feed/ws.go`) streams `UserFills` to the live executor for fill reconciliation (live mode only). Fill callback routes to `FillDecay`, `FillRateTracker`, `MarkoutTracker`, and `SideToxicityTracker`.
4. **Spread engine** (`spread/engine.go`) pairs the first two configured assets (`Assets[0]` / `Assets[1]`), computes raw z-score and OLS beta via circular buffer, emits `SpreadSignal` to quote manager. Only instantiated when `spread_engine_enabled` is true (defaults to auto: on with 2+ assets, can be explicitly disabled).
5. **Quote manager** (`quoting/manager.go`) runs a multi-step pipeline per asset, then calls `Executor.Refresh()` to cancel-and-replace orders:
   1. **Fair value** -- microprice-anchored FV (optionally multi-level via `microprice_levels`), blended with short VWAP (`vwap_alpha` controls mix)
      1.6. **Trade-flow imbalance shift** -- shifts FV by trade imbalance × `trade_imbalance_max_shift_bps`
      1.7. **Post-fill adverse decay** -- shifts FV by exponentially decaying post-fill shift (`FillDecay`)
      1.8. **Markout-to-FV feedback** -- shifts FV based on per-side markout EWMA asymmetry (`markout_fv_scale`); when buys are more toxic, FV shifts down to widen bids and tighten asks
   2. **Base quotes + microstructure** -- half-spread around fair value (+ optimizer spread delta); widened by warmup/recovery multipliers, microstructure width scaling (`dynamic_width`)
      2.5. **Fee floor** -- when `fee_floor_enabled`, widens quotes to cover round-trip maker + builder fees
   3. **Spread bias** -- tanh-shaped continuous shift from z-score (`spread_bias_scale`, `spread_bias_max_bps`); supports asymmetric allocation (`spread_bias_asymmetry`) and self-calibrating z-score (Feature 5)
   4. **Inventory skew** -- convex (`inventory_skew_gamma`), asymmetric (`inventory_skew_asymmetry`) skew toward reducing position; when `inventory_skew_exit_preserve` is true, only shifts the entry side (A1). Magnitude from `InventoryController` half-life (B3) × optimizer skew_mult (C)
      4.5. **Cross-leg inventory skew** -- beta-normalized cross-leg skew for correlated pairs (`cross_leg_skew_enabled`)
   5. **VWAP wick sizing** -- per-side size multipliers when mid deviates from both short and long VWAPs; leans into mean reversion with tighter spacing. Also vol-scaled ladder spacing.
   6. **Z-score widening** -- asymmetric (`z_widening_asymmetry`) progressive widening when |z| exceeds threshold, with self-calibrating z (Feature 5)
      6.5. **Per-side toxicity widening + size scaling** -- when `toxicity_per_side` is true, independently widens bid/ask and scales notional based on per-side markout EWMAs (B2). Replaces legacy symmetric adverse selection.
      6.9. **Spread stack cap** -- caps total per-side spread at `max_total_spread_multiplier × base_spread`, preventing multiplicative compounding of toxicity + burst + optimizer from creating unfillable quotes
   7. **Crossing protection** -- position-aware: keeps exit-side order at its computed price, pulls entry side to maintain `min_crossing_spread_bps` minimum spread (default 2 bps). Falls back to midpoint clamp when flat.
   - **Sizing** -- applies adaptive notional multiplier (Feature 2 `FillRateTracker`) × per-side fill rate multiplier × optimizer notional_mult (C) × per-side toxicity size factor (B2), warmup/recovery/degraded scaling, beta hedge scaling, pair-reduce mode
   - **Exposure limits** -- clamps to `max_position_notional` / `max_spread_exposure`
6. **Wall-fronting** -- when `wall_front_enabled`, snaps the nearest ladder level in front of large resting orders (walls), detected by `wall_min_size_multiplier` × average book level size. Applied during ladder construction in `buildSideOrders()`.
7. **Risk manager** (`risk/manager.go`) monitors position notional and equity drawdown, emits `RiskAction` (HALT/RESUME). Tracks consecutive halt count (`max_consecutive_halts`) and requires equity recovery to `min_resume_equity_pct` of high-water before resuming
8. **Executor** (`execution/executor.go` interface) -- `PaperExecutor` simulates fills from book updates; `LiveExecutor` wraps SDK with 3-retry exponential backoff (orders route via QuickNode, incur builder fee)

### Spread Engine Sampling

The spread engine only samples when **both** legs have fresh data since the last successful sample. `Assets[0]` updates are buffered; a sample fires when `Assets[1]` updates while `Assets[0]` is still dirty (and vice-versa), gated by `max_cross_leg_skew_ms` (default 150 ms). Z-score is computed against the current window _before_ adding the new value. Which asset is which leg is determined purely by ordering in `strategy.assets` — nothing is hardcoded to CL/BRENTOIL despite historical comments.

### Channel Fan-out Pattern

Simplified wiring: `signalCh` is direct (single consumer: quote manager), `posCh` is direct (single consumer: risk manager). `bookCh` uses fan-out via variadic channels passed to `feed.NewGRPCFeed`: spread engine (if 2+ assets), quote manager, live executor (if live mode), or paper executor (if paper mode). Trade updates fan out to per-asset short VWAP, long VWAP, and trade-flow imbalance calculators via a goroutine in `main.go`. Fill callbacks route to `FillDecay`, `FillRateTracker`, `MarkoutTracker`, and `SideToxicityTracker` (all initialized inside the quote manager).

## Configuration

Config path is supplied via `-config <file>` (no implicit default). Current configs:

- `config.yaml` — runtime config; created by copying `config.example.yaml` and editing. Not committed.
- `config.example.yaml` — annotated template, **single-asset (`xyz:CL`) by default**; source of truth for the config schema. Every field is inline-documented.

Top-level sections in the YAML schema:

- `strategy` — asset selection + spread-engine knobs (cross-leg z-score, beta bounds)
- `market_making` — quoting, sizing, signals, guards, optimizer v2
- `risk` — position / drawdown / session-loss limits
- `execution` — paper vs. live, leverage, polling cadence
- `logging` — verbosity + periodic status cadence
- `rate_limit` — API-budget circuit breaker (optional, advanced)
- `postmortem` — JSON snapshot dumps on halt (optional, advanced)

Environment variables (`QUICKNODE_SUBDOMAIN`, `QUICKNODE_AUTH_TOKEN`, `PRIVATE_KEY`) override YAML `env:` values. Short subdomain names are auto-completed with `.hype-mainnet.quiknode.pro`.

Two execution modes:

- **Paper** (`live_mode: false`): no private key needed, simulates fills when book price crosses order price
- **Live** (`live_mode: true`): requires `PRIVATE_KEY`, sets leverage at startup, cancels all orders on shutdown. Orders route through the QuickNode endpoint via the SDK (incurs builder fee).

The `-validate` flag loads + validates a config and exits. Credential checks are skipped so it works offline — `make validate` uses this to smoke-test `config.example.yaml` in CI.

All `market_making`, `strategy`, and `risk` fields are documented inline in `config.example.yaml` (and in `docs/CONFIG.md`). Read those for the canonical schema; only non-obvious cross-cutting behaviors are kept here.

Non-obvious behaviors worth knowing before touching:

- **Optimizer per-asset overrides**: `optimizer_v2.per_asset` is struct-level replace, not field merge — the override MUST include `enabled: true` or the asset silently falls off the optimizer.
- **`inventory_skew_exit_preserve`** (A1): when true, skew only shifts the entry side; exit side quotes at natural price.
- **Toxicity attenuation** (B2): per-side toxicity widening is attenuated when the optimizer has already widened the same side, to avoid double-counting. Output is also suppressed until `toxicity_warmup_fills` is reached.
- **Spread stack cap** (`max_total_spread_multiplier`): caps compounded per-side spread so toxicity + burst + optimizer can't produce unfillable quotes.
- **Crossing protection**: position-aware — keeps exit-side at its computed price and pulls only the entry side to maintain `min_crossing_spread_bps`. Midpoint clamp is the flat fallback.
- **Rate-limit allocator**: when `rate_limit_allocator_enabled`, API budget is distributed per-asset by ROI, not uniformly. `VirtCycleEngine` feeds `BudgetPressure` to optimizer v2.
- **Flow Gate `min_util`**: scales adverse-flow suppression by inventory utilization, so the gate is a no-op when nearly flat (default 0.25).

### Per-Asset Config Overrides

Each of `market_making`, `execution`, and `risk` sections supports a `per_asset` map that overrides specific fields for individual assets. Unspecified fields inherit from the global (top-level) config. This allows running multiple assets with different parameters in a single bot process.

- `strategy.spread_engine_enabled` -- explicit bool to control spread engine. `nil` = auto (enabled with 2+ assets), `false` = disabled (for independent assets like NVDA + MU)
- `market_making.per_asset` -- override any MM field (spread, notional, ladder, VWAP, etc.) per asset
- `execution.per_asset` -- override `leverage` per asset
- `risk.per_asset` -- override `max_position_notional`, `max_spread_exposure`, `max_open_orders` per asset

The `Config.ForAsset(coin)` method returns the fully resolved config for a given asset. All runtime code uses `ForAsset()` instead of reading global config directly.

Removed fields (from previous architecture): `entry_z_score`, `exit_z_score`, `stop_z_score`, `max_z_score`, `ExitConfig`, `hedge_timeout_ms`, `max_hedge_slippage_bps`, `spread_circuit_breaker_bps`, `adaptive_spread_enabled`, `adaptive_spread_k`, `adaptive_skew_enabled`, `inventory_skew_min`, `inventory_skew_max`, `adaptive_alpha_enabled`, `vwap_alpha_min`, `vwap_alpha_max`, `adverse_selection_enabled` (renamed: `adverse_selection_half_life_sec` → `toxicity_half_life_sec`, `adverse_selection_max_widen` → `toxicity_max_widen`, `adverse_selection_baseline_bps` → `toxicity_baseline_bps`), `RuntimeOptimizerConfig`, `OptimizerThresholdsConfig`, `OptimizerInputs`, `RuntimeOptimizer`, `KnobBoundsConfig.Step`, `KnobBoundsConfig.CooldownSec` (v1 optimizer consolidated into v2).

## Hyperliquid Wallet Structure

Hyperliquid has three separate balances: **Spot**, **Perps**, and **HIP-3 Dex**. USDC deposited to Hyperliquid lands in the Spot wallet. It must be transferred to Perps (or a HIP-3 dex) before it shows as equity in `ClearinghouseState`. The `Equity()` method sums dex `marginSummary.accountValue` + spot free USDC (`total - hold`) for dex traders (2 API calls), or perps `marginSummary.accountValue` + spot USDC for non-dex traders.

- `sdk.Info().ClearinghouseState(addr)` — perps balance (`marginSummary.accountValue`)
- `sdk.Info().ClearinghouseState(addr, WithDex("xyz"))` — HIP-3 dex balance
- `sdk.Info().SpotClearinghouseState(addr)` — spot balances (`balances[].total` where `coin == "USDC"`)

## SDK

Uses `github.com/quiknode-labs/hyperliquid-sdk/go/hyperliquid` (v0.1.7). Key SDK calls:

- `sdk.Buy()` / `sdk.Sell()` with `WithSize`, `WithPrice`, `WithTIF(TIFALO)` (post-only / anchor limit order)
- `sdk.Modify(oid, coin, side, price, size, ModifyWithTIF(...))` for in-place order modification (cancel+replace)
- `sdk.Cancel(oid, coin)` for cancelling individual HIP-3 orders by OID
- `sdk.CancelAll(coin)`, `sdk.ClosePosition(coin)`, `sdk.UpdateLeverage(coin, leverage, ...opts)`
- `sdk.Markets()` for szDecimals (searches Perps, Spot, then HIP3 market lists)
- `sdk.Info().Meta()` for exchange metadata (max leverage per asset)
- `sdk.Info().UserRateLimit(address)` for API rate limit status
- `sdk.Info().ClearinghouseState(address)` for perps equity
- `sdk.Info().ClearinghouseState(address, WithDex(dex))` for HIP-3 dex equity
- `sdk.Info().SpotClearinghouseState(address)` for spot balances
- `sdk.Info().OpenOrders(address)` for live order queries
- `hyperliquid.IsRetryable(err)` for retry decisions
- `hyperliquid.NewGRPCStream()` / `hyperliquid.NewStream()` for data feeds
- `hyperliquid.LeverageWithIsolated()` -- option for isolated margin (HIP-3 assets)
- `hyperliquid.L2BookNLevels(n)` -- limit L2Book gRPC subscription depth
- `stream.L2Book(coin, callback, ...opts)` -- gRPC L2Book subscription
- `stream.Trades(coins, callback)` -- gRPC public trade subscription

## Reference Documentation

- QuickNode Hyperliquid docs: https://www.quicknode.com/docs/hyperliquid.md
- QuickNode Hyperliquid LLM reference: https://www.quicknode.com/docs/hyperliquid/llms.txt
- QuickNode Hyperliquid API overview: https://www.quicknode.com/docs/hyperliquid/api-overview.md
- hyperliquidapi.com docs: https://hyperliquidapi.com/ + https://hyperliquidapi.com/llms.txt
- Go SDK source: https://github.com/quiknode-labs/hyperliquid-sdk (Go SDK at `/go`)
- Hyperliquid API rate limits: https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api/rate-limits-and-user-limits
  - Please review the doc when doing anything related to rate limits or asked about it
  - The rate limiting logic allows 1 request per 1 USDC traded cumulatively since address inception
  - **Rate limit `remaining` / `cap` / `pct` are cumulative since address inception — not per-session.** A session that opens at "4.5% remaining" just means prior sessions/trading have used most of the lifetime budget; it is NOT a sign this session has already burned rate limit. `VirtCycle` regenerates budget from cumulative USDC volume, also since inception. When analyzing logs, don't reason about rate-limit pct as if it resets on startup.

## Conventions

- Structured JSON logging via `log/slog` (no third-party logger)
- Paper executor log lines prefixed with `[PAPER]`
- All shared types in `internal/types/types.go`
- Executor interface in `execution/executor.go`; paper and live are separate files
- VWAP calculator in `vwap/calculator.go`
- Trade-flow: `tradeflow/calculator.go` (rolling imbalance), `tradeflow/fill_decay.go` (post-fill FV decay), `tradeflow/saturation.go`, `tradeflow/volume_clock.go`
- Quoting sub-components:
  - Signals: `regime.go` (lag-1 autocorrelation), `microstructure.go`, `velocity.go` (volume burst detection), `perside_stats.go`, `side_presence.go`, `quantile_normalizer.go`, `wall_quality.go`, `price_shock.go`, `shock_chain.go`
  - Fill/markout: `fill_rate.go` (per-side EWMA + `RecordFillSide`/`NotionalMultiplierSide`), `fill_burst.go`, `markout.go` (500ms/1s/2s/5s horizons), `toxicity.go` (per-side EWMA markout + FV feedback), `funding.go`
  - Controllers: `inventory_controller.go` (half-life-based adaptive skew), `optimizer_overlay.go`, `optimizer_v2.go` (4D state model, policy table, proportional control), `virt_cycle.go` (rate-limit regen), `rate_limit_allocator.go` (per-asset ROI budget), `supervisor.go` (halt/degrade/recovery state machine), `session_pnl.go`, `postmortem.go`
- Config system: `config/config.go` (YAML loading, validation, topology enforcement), `config/merge.go` (reflection-based per-asset override merge — replaces handwritten field-by-field apply functions)
- Data feeds: `feed/grpc.go` (L2Book), `feed/trades_grpc.go` (gRPC public trades for HIP-3), `feed/ws.go` (UserFills)
- Execution plumbing: `execution/executor.go` (Executor interface), `execution/transport.go` (Transport interface implemented by `sdk_transport.go`)
- L2Book parser (`feed/grpc.go`) handles multiple SDK data formats with cascading fallbacks
- Monitor TUI: `cmd/monitor/` — separate binary, pure log consumer (no imports from `internal/`). Uses bubbletea v2 + lipgloss v2. Key files: `state.go` (state types + ring buffer), `parser.go` (JSON log dispatch), `tailer.go` (fsnotify + poll file watcher), `model.go` (bubbletea Model), `view*.go` (panel renderers), `styles.go` (lipgloss theme)

---
> Source: [zer0cache/hyperliquid-market-maker-bot](https://github.com/zer0cache/hyperliquid-market-maker-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
