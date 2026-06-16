## foxml-trader-v2

> Tick-level crypto trading engine in C++. Branchless fixed-point arithmetic, bitmap-based portfolio management, regression-driven gate adjustment with regime detection. Per-core sharded hot path (40-400ns p99), single-symbol producer thread fanning real Binance ticks across SPSC rings to per-core consumers.

# CLAUDE.md

## Overview

Tick-level crypto trading engine in C++. Branchless fixed-point arithmetic, bitmap-based portfolio management, regression-driven gate adjustment with regime detection. Per-core sharded hot path (40-400ns p99), single-symbol producer thread fanning real Binance ticks across SPSC rings to per-core consumers.

**Sharded is production. Legacy single_core LIVE is deprecated (warned at boot). Legacy backtest is gone — `Backtest_Run` wraps `BacktestSharded_Run`.**

## Build

`./build.sh test` (engine + controller_test), `gui` (engine_gui + foxml_suite), `suite` (suite with XGBoost), `tsan` / `asan` (sanitizer builds), `all`, `clean`. Build flags: `-DLATENCY_PROFILING=ON`, `-DLATENCY_LITE=ON`, `-DLATENCY_BENCH=ON`, `-DBUSY_POLL=ON`, `-DUSE_NATIVE_128=ON`.

Build dirs (different compile flags → different outputs): `build/` (ANSI + tests, zero deps), `build_gui/` (engine_gui + foxml_suite — SDL2 + OpenGL3 + ImGui), `build_suite/` (same + XGBoost), `build_lat/` (LATENCY_PROFILING), `build_tsan/`, `build_asan/`.

XGBoost C library (for `-DUSE_XGBOOST=ON`): clone `dmlc/xgboost` recursive, cmake with `-DBUILD_STATIC_LIB=OFF`, install + ldconfig.

`build.sh` symlinks `engine.cfg` into each build dir; `bin/engine_gui` → `build_gui/engine_gui`.

## Architecture (sharded)

N cores (default 4, cap 16), each = self-contained strategy unit (slow + hot pthread pair). Producer thread fans Binance ticks across per-core SPSC rings; per-core hot-paths consume; per-core slow-paths rebuild gate parameters on cadence.

- Per-core strategy (`core_N_strategy=simple_dip|momentum|ema_cross|ml`)
- Per-core ML model (`core_N_model_path=...` or `core_N_model_dir=...`)
- Per-core risk (`core_N_risk_pct=...`)
- Per-core ConfidenceScorer (when STRATEGY_ML)
- Per-core slow_state owns rolling/regime/flow data (v5.1.2+)
- Hot path p99 ≤500ns
- Partial exits (`partial_exit_enabled=1`): each core owns 2 slots (legs A+B); max cores = 8
- `engine_arch=per_core_slow` (default v5.0+) | `centralized` (legacy)

```
HOT PATH (per tick, per core, branchless):
  BG_Evaluate → SG_Evaluate ×2 → TradeEvent push (rare branch)

SLOW PATH (per-core thread, every poll_interval ticks):
  EventLoop_UpdateRollingStateOneCore → RebuildOneCore (regime + strategy)
  → ExecutionCore_SetParameters (seqlock to hot path)
  → TimeExitOneCore → TrailingSLRatchetOneCore

GLOBAL THREADS:
  Producer: tick read + fan_out + ema_price replication + GUI publish
  Drainer:  OMS_DrainSubmit + OrderManager_Tick + DrainPostFill
  Async:    Binance trade WS, depth WS, Tick/DepthRecorder, Notify worker, GUI
```

## Data Flow: Regime Detection

```
Per-engine slow_state (RollingStats × 4 + RORRegressor + flow + depth) →
  RegimeSignals (slope, R², variance, ror_slope, ema_sma_spread,
                 book_imbalance, spread_z, flow_*_ewma, large_trade_z, ...) →
  Regime_Classify (trending_score + volatile_score with hysteresis) →
    RANGING / TRENDING / VOLATILE / MILD_TREND → strategy dispatch
```

`RegimeSignals` is the extensibility point — see `DOCS/CLAUDE_INTEGRATION.md` for the recipe.

## Directory Structure

- **CoreFrameworks/** — OrderGates, Portfolio, ExecutionCore, ControllerEventLoop, EngineSharded, ShardedSnapshot/Persist, GateParameters, TradeEvent, OrderManager, ShardedBacktestDriver
- **Strategies/** — RegimeDetector, MeanReversion, Momentum, SimpleDip, EmaCross, MLStrategy, StrategyParameters (dispatcher), StrategyInterface
- **DataStream/** — BinanceCrypto/Depth, DepthReplayState, DepthRecorder, TickRecorder, BinanceOrderAPI, EngineTUI
- **FixedPoint/** — FPN<F=64> (4096-bit)
- **MemHeaders/** — PoolAllocator (bitmap order pool), BuddyAllocator
- **ML_Headers/** — RollingStats, ROR_regressor, ConfidenceScore, ModelInference (XGBoost), FlowFeatures
- **GUI/** — Dear ImGui native: FoxmlTheme, DashboardPanels, ChartPanel, CandleAccumulator, TradeReader, SettingsPanel, TradeHistoryPanel, LogViewerPanel, GuiThread
- **Backtest/** — `Backtest_Run` wrapper + `BacktestSharded_Run`, BacktestPanels, LabelFunctions, HeldOutSplit, ValidationSplit
- **tests/** — controller_test.cpp (739+ assertions), parity_harness.cpp
- **DOCS/** — CHANGELOG.md, changelogs/, CLAUDE_*.md (split-load reference docs)
- **plans/** — gitignored, working plans
- **Version.hpp**, **Limits.hpp** — single source of truth

## Code Conventions

- `using namespace std;` throughout
- C-style with templates, no classes (with one explicit exception:
  RAII destructors on resource-owning structs that own threads or
  mmap'd memory, e.g. `~OrderManagerState()` since v5.11.26 — see
  the destructor comment in `CoreFrameworks/OrderManager.hpp` for
  the criteria; default is still no destructor)
- `Pattern_FunctionName` (e.g. `Portfolio_Init`, `BG_Evaluate`)
- Hot-path math is `FPN<F>` only, no floats (F=64 = 4096-bit)
- Branchless: mask tricks `-(uint64_t)pass`, word-level mask-select
- Inline comments explain reasoning, not what
- **Preserve user's voice in existing comments when editing**

### Test file size discipline (added v5.11.35)

`tests/controller_test.cpp` is currently ~16k lines (1822 tests).
That's too big — slow to compile, hard to navigate, easy to break
during refactors. The build system already supports multiple test
binaries (`depth_recorder_test`, `parity_harness` are precedents).

**Rule:** any test file > 5k lines OR > 100 test sections must be
split BEFORE adding more tests. Categories should be domain-aligned:

  - `controller_test_engine.cpp` — engine paths, OMS, drainer, gates
  - `controller_test_features.cpp` — RollingStats, FeatureRegistry,
    Features_PackAll, FeatureStandardizer
  - `controller_test_stamps.cpp` — verify_model_stamp,
    stamp_write_for_model, bash-parity, scaler binding
  - `controller_test_ml.cpp` — ConfidenceScore, MLBuildContext,
    CoreModelZoo, BanditLearning, FOREACH_FEATURE
  - `controller_test_misc.cpp` — utility tests, FPN math, allocators

Helpers (`check`, `tests_passed`, `tests_failed`, `make_event`,
`fpn_field_eq` etc.) extract into `tests/test_common.hpp`. Each
split file is a separate `add_executable()` in CMakeLists.txt;
all run via `ctest` or the existing `./build.sh test` harness.

The actual split for the current `controller_test.cpp` is queued
as a v5.11.35 sub-ship (deferred from the current session because
1822 tests at risk warrants a focused effort with rollback anchor).

## Key Design Decisions

1. Portfolio uses `uint16_t` bitmap (not sequential count)
2. Per-position TP/SL exits on hot path, portfolio mgmt on slow path
3. Fill consumption every tick (zero unprotected exposure)
4. Per-core data-plane: each engine owns its rolling/regime/flow state (v5.1.2+)
5. OMS submit funneling: drainer is sole `OrderManager_Submit` caller (v4.7.37)
6. OneCore helpers shared by 3 callers (centralized live, per_core_slow live, backtest) for structural train-serve parity
7. Warmup observes market before trading (gates on slow-path sample count)
8. TUI independent of engine (engine runs headless, TUI reads state via double-buffered TUISnapshot)
9. No API key for market data WS (public endpoint)
10. Partial exits: dispatcher post-cap so strategies stay leg-A-only; hot path branch-gates leg B
11. Smart CPU pinning (v5.1.5): slow-paths avoid SMT siblings of busy threads via /sys topology read
12. Display ↔ execution invariant (v5.6.0): every term in BG_Evaluate / SG_Evaluate must have a corresponding GUI surface. Adding a new hot-path predicate term requires a `PerCoreSnap` field + a panel render in the same PR. See `DOCS/EXECUTION_DISPLAY_INVARIANTS.md`.
13. X-macro registry is the standard pattern for multi-site additions (v5.8.0+). Any category where "adding the next instance" requires touching ≥2 code sites must use a `FOREACH_<CATEGORY>(X)` registry. See `DOCS/EASY_ADDITIONS_INVARIANTS.md` for the canonical spec + the audited categories. Instances: strategies, ML features, SHALT codes, halt_reason codes, regimes, stateful GUI panels, backtest metrics.
14. NaN-free feature pack (v5.9.0+). `Features_PackAll` is the single chokepoint where every feature value is validated. Two-layer guard: `FPN_IsValidFinite` (catches FPN saturation past 1e15) + IEEE-754 `isnan/isinf` post float-cast. Returns `-1` sentinel on any failure; caller skips the prediction cycle and increments `nan_feature_events_total` (distinct from `nan_prediction_events_total`). Adding a new feature must NOT add a separate validation site — pack-time is the load-bearing surface, downstream code trusts the pack output. See `DOCS/CLAUDE_ML_INVARIANTS.md`.
15. Parity-tested-by-construction (v5.9.5+). Train-serve parity surfaces (features, labels, scaler, cfg, stamp body, threading, build flags) gain protection by adding a registry/binding/snapshot rather than ad-hoc tests. Pattern: FEATURE_REGISTRY_HASH (v5.8.6) + scaler `feature_registry_hash` (v5.9.3a) + stamp body `has_*` forward-compat flags (v5.9.3a, v5.9.4a) + snapshot tests for compute-fn bodies (v5.9.2a). Prefer Surface G stamp body extension (`has_<field>` flag with `model_format_version` UNCHANGED) over `MODEL_FORMAT_VERSION` bumps — flag extensions don't break legacy stamps; format bumps lose data. Run `/parity-check` before declaring an ML-side sprint complete. See `DOCS/PARITY_LIFECYCLE.md` + `DOCS/PARITY_VERIFICATION_CHECKLIST.md`.

---

# Reference Docs (split-load — read on demand)

To keep this CLAUDE.md small (always loaded), detailed references live in
separate files. **Read the relevant file when starting work in that area:**

| When working on... | Read |
|---|---|
| Adding a cfg field, GUI panel, strategy, ML feature, per-core override | [`DOCS/CLAUDE_INTEGRATION.md`](DOCS/CLAUDE_INTEGRATION.md) |
| Changing OMS, kill switch, snapshot, fee math, hot path, slow-path threading, anything load-bearing | [`DOCS/CLAUDE_INVARIANTS.md`](DOCS/CLAUDE_INVARIANTS.md) |
| Touching FeatureRegistry, FeatureComputeCtx, MLBuildContext, CoreModelZoo, stamp_*, verify_model_stamp, train→serve path | [`DOCS/CLAUDE_ML_INVARIANTS.md`](DOCS/CLAUDE_ML_INVARIANTS.md) |
| Planning a multi-day change | [`DOCS/CLAUDE_REVIEW.md`](DOCS/CLAUDE_REVIEW.md) |
| Backtest suite (Run Control, Training, Walk-Forward, Held-Out) | [`DOCS/CLAUDE_FOXML_SUITE.md`](DOCS/CLAUDE_FOXML_SUITE.md) |

These are *never* automatically loaded — Claude reads them on-demand when
the conversation matches one of the rows above. Keeps the always-on
context small (~1500 words) so routine changes are fast, while the
detailed rules are still authoritative when needed.

---
> Source: [Jennyfirrr/FoxML_Trader_v2](https://github.com/Jennyfirrr/FoxML_Trader_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
