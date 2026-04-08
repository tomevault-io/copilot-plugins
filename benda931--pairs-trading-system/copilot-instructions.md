## pairs-trading-system

> handles TTL caching, fallback order (CHAMPION → CHALLENGER → CANDIDATE → neutral),

# CLAUDE.md — Pairs Trading System

This file is read automatically by Claude Code at the start of every session.
It captures architecture decisions, coding conventions, and context that would
otherwise require re-explaining each time.

---

## What This System Is

An institutional-grade **statistical arbitrage platform** for equities pairs trading.

**North star:** Portfolio-level, market-neutral, regime-aware, factor-aware relative-value
platform where correlation/distance are used for discovery, cointegration/stability/half-life
for validation, and residual mean reversion is the primary alpha abstraction.

## Current Integration Status (as of 2026-04-01)

> **IMPORTANT — READ BEFORE CONTRIBUTING**
>
> This repository contains two distinct software systems:
>
> **System A (Operational):** `core/`, `root/`, `scripts/`, `common/` — actively used for backtesting,
> parameter optimization, and Streamlit dashboards. These are real, tested, and produce meaningful results.
>
> **System B (Infrastructure):** `research/`, `portfolio/`, `ml/`, `agents/`, `runtime/`, `governance/`,
> `orchestration/`, `audit/`, `surveillance/`, `attestations/`, `controls/`, `policies/`, `retention/`,
> `reporting/` — professionally designed, comprehensively tested in isolation, with **partial integration
> with System A** (see status below). These packages exist as validated infrastructure; most awaiting
> further integration.
>
> **Integration status:**
> - ML platform: designed and tested; zero models trained; ModelScorer never called from signal path
> - **Agents: 2 of 40 agents dispatched from operational code.**
>   `SystemHealthAgent` dispatched after `health_check`; `DataIntegrityAgent` dispatched after `data_refresh`.
>   Both via `PairsOrchestrator.run_daily_pipeline()` → `monitoring/workflow.py` (WorkflowEngine),
>   BOUNDED_SAFE, no approval gate, alert bus integration, direct-dispatch fallback.
>   38 remaining agents are registered in the registry but dispatched from nowhere (scaffold-only).
>   See `monitoring/workflow.py`, `core/orchestrator.py:run_agent_data_integrity_check()`.
> - Portfolio allocator: `bridge_signals_to_allocator()` called from `run_daily_pipeline()` via
>   `run_portfolio_allocation_cycle()` on every operational run (P1-PORTINT COMPLETE).
>   Dashboard UI panel in `root/portfolio_tab.py` also allows manual dispatch.
> - Runtime/control plane: `is_safe_to_trade()` injected as safety_check in every `run_portfolio_allocation_cycle()` call (P1-SAFE COMPLETE).
>   Kill-switch: `make_kill_switch_manager_with_control_plane()` called per cycle (P0-KS COMPLETE).
> - CLI pipeline trigger: `python scripts/run_daily_pipeline.py` runs the full pipeline end-to-end.
>   Daemon mode: `start_daemon()` on PairsOrchestrator schedules daily runs (APScheduler or threading.Timer fallback).
> - Governance/audit: implemented; no operational decision is gated by any governance check
> - Walk-forward: `root/optimization_tab.py` does calendar stability validation (not true WF); UI now labeled
>   "Temporal Stability Check — In-Sample Calendar Segments" with ADR-007 disclaimer (P0-WF COMPLETE).
>   True walk-forward lives in `research/walk_forward.py` and is not yet wired to the optimizer.
>
> **See:** `docs/remediation/remediation_ledger.md` for the formal integration status register.
>
> **See:** `docs/adr/ADR-007-backtest-realism-limitations.md` for documented backtest assumptions.

---

## Directory Layout

```
pairs_trading_system/
├── core/                    # Domain logic (no UI, no IO)
│   ├── contracts.py         # ← THE single source of truth for all types/enums
│   ├── fmp_fundamentals.py  # FMP fundamentals engine
│   ├── fmp_macro.py         # FMP macro data engine
│   ├── leverage_engine.py   # Kelly sizing, vol target, risk parity
│   ├── tail_risk.py         # Cornish-Fisher ES, VaR backtest
│   ├── trade_monitor.py     # Trade health scoring, exit signals
│   ├── attribution.py       # MC = SDS*(1-FJS)*(1-MSS)*(1-STF) framework
│   ├── ml_validation.py     # Purged K-fold CV, PSI drift detection
│   ├── intents.py           # Intent objects (EntryIntent, ExitIntent, SignalDecision, ...)
│   ├── diagnostics.py       # SpreadStateSnapshot, SignalDiagnostics, ExitDiagnostics, SignalAuditRecord
│   ├── lifecycle.py         # TradeLifecycleStateMachine (13 states), LifecycleRegistry, CooldownPolicy
│   ├── threshold_engine.py  # ThresholdEngine: STATIC/VOL_SCALED/REGIME_CONDITIONED modes
│   ├── signal_quality.py    # SignalQualityEngine: grade A+→F, size multipliers, meta-label hook
│   ├── regime_engine.py     # RegimeEngine: waterfall + ML hook, RegimeFeatureSet, tradability modifiers
│   ├── signal_analytics.py  # SignalAnalytics, LifecycleAnalytics, ExitAnalytics
│   └── ...                  # Existing: optimizer, backtester, signals, etc.
│
├── research/                # Offline research pipeline (train-time only)
│   ├── discovery_contracts.py  # Discovery-layer domain types (InstrumentMetadata, CandidatePair, etc.)
│   ├── universe.py          # Universe construction, eligibility filtering, composition analytics
│   ├── candidate_generator.py  # Multi-family candidate generation (correlation, distance, cluster, coint)
│   ├── stability_analysis.py   # Rolling stability, structural break detection, regime suitability
│   ├── discovery_pipeline.py   # End-to-end discovery orchestration, scoring, ranking
│   ├── pair_validator.py    # Statistical validation with explicit rejection log
│   ├── spread_constructor.py # OLS / Rolling OLS / Kalman spread fitting
│   └── walk_forward.py      # Walk-forward harness with purged splits
│
├── portfolio/               # Capital allocation & risk operating model
│   ├── contracts.py         # Portfolio domain types (RankedOpportunity, SizingDecision, AllocationDecision, ...)
│   ├── capital.py           # CapitalManager — pool, sleeves, allocation lifecycle
│   ├── ranking.py           # OpportunityRanker — 7-dimension composite scoring
│   ├── sizing.py            # SizingEngine — vol-target + scalar stack → SizingDecision
│   ├── exposures.py         # ExposureAnalyzer — sector/cluster/shared-leg
│   ├── allocator.py         # PortfolioAllocator — main cycle engine
│   ├── risk_ops.py          # DrawdownManager + KillSwitchManager (4-layer risk model)
│   └── analytics.py         # PortfolioAnalytics — snapshot, diagnostics, attribution
│
├── agents/                  # Agent infrastructure
│   ├── base.py              # BaseAgent, AgentAuditLogger
│   ├── registry.py          # AgentRegistry, dispatch, permission system
│   ├── signal_agents.py     # SignalAnalystAgent, RegimeSurveillanceAgent, TradeLifecycleAgent, ExitOversightAgent
│   └── portfolio_agents.py  # PortfolioConstructionAgent, CapitalBudgetAgent, ExposureMonitorAgent,
│                            #   DrawdownMonitorAgent, KillSwitchAgent, DeRiskingAgent
│
├── models/                  # ML infrastructure
│   ├── base.py              # ModelProtocol, ModelMetadata, ModelRegistry
│   ├── dataset_builder.py   # Leakage-safe dataset construction
│   └── feature_store.py     # Named feature definitions + compute
│
├── common/                  # Shared utilities
│   ├── fmp_client.py        # FMP HTTP client (stable-first, retry, cache)
│   └── data_providers.py    # Multi-source data provider with priority routing
│
├── root/                    # Streamlit dashboard tabs
├── docs/                    # Architecture documentation
│   ├── architecture.md
│   ├── discovery_methodology.md
│   ├── signal_architecture.md  # Signal/regime/lifecycle/threshold/quality/agent layer
│   ├── portfolio_architecture.md  # Portfolio construction, capital, sizing, risk ops
│   └── adr/                 # Architecture Decision Records
│
└── tests/                   # Test suite
```

---

## Critical Conventions

### 1. core/contracts.py is the source of truth
**Never define domain types anywhere else.** All enums, dataclasses for PairId,
SpreadDefinition, AgentTask, AgentResult, ValidationResult, etc. live here.
Import from `core.contracts` everywhere.

### 2. Train/Test boundary is sacred
Every function that estimates parameters takes a `train_end: Optional[datetime]`
argument. Parameters are **always** estimated strictly from `data[data.index <= train_end]`.
The word "fit" means training data only. The word "transform" means applying
pre-fitted parameters to any window (including test).

### 3. Rejection reasons are always explicit
`PairValidationReport` always carries a populated `rejection_reasons` list when
`result == ValidationResult.FAIL`. Never fail silently.

### 4. FMP API conventions
- Use the `/stable/` prefix for all endpoints
- Symbol is **always a query parameter**: `?symbol=AAPL` (not a path segment)
- Stable endpoints that work: `historical-price-eod/full`, `ratios-ttm`,
  `key-metrics-ttm`, `income-statement`, `balance-sheet-statement`,
  `cash-flow-statement`, `income-statement-growth`, `earnings`,
  `economic-calendar`, `treasury-rates`
- Field names: `priceToEarningsRatioTTM` (not `peRatioTTM`), `evToEBITDATTM`,
  `incomeQualityTTM`, `epsActual`/`epsEstimated` for earnings surprises

### 5. Agents have narrow mandates
Each agent does exactly one job. They accept `AgentTask`, return `AgentResult`,
and call `audit.log()` for every significant decision. Never mutate shared state
inside an agent — return a result and let the caller decide what to do with it.

### 6. ML leakage prevention
`DatasetBuilder` enforces: features at time T use data up to T, labels use data
after T. `BaseModel.fit()` raises `ValueError` if X contains timestamps after
`train_end`. `PurgedKFold` (in `core/ml_validation.py`) implements de Prado-style
purge + embargo for cross-validation.

---

## Key Thresholds (ValidationThresholds in contracts.py)

| Check | Default | Hard/Soft |
|-------|---------|-----------|
| Min correlation | 0.60 | Hard |
| Max ADF p-value | 0.10 | Hard |
| Min cointegration p-value | 0.10 | Hard |
| Min half-life (days) | 2 | Hard |
| Max half-life (days) | 120 | Hard |
| Min Hurst exponent | 0.0 (mean-reverting) | Soft |
| Max Hurst exponent | 0.45 | Soft |
| Min rolling hedge ratio stability | 0.30 | Soft |

---

## Data Sources (Priority Order)

1. **IBKR** (priority 10) — live/paper trading data
2. **FMP** (priority 25) — prices, fundamentals, macro
3. **Yahoo Finance** (priority 50) — fallback

FMP API key: stored in config or environment as `FMP_API_KEY`.

---

## Walk-Forward Configuration Defaults

```python
WalkForwardHarness(
    n_splits=5,
    test_days=252,        # 1 year test window per fold
    min_train_days=504,   # 2 years minimum training
    embargo_days=20,      # 20-day gap to prevent leakage
)
```

---

## Common Gotchas

- `PairId` always stores symbols in **lexicographic order** (sym_x < sym_y).
  Use `PairId("AAPL", "MSFT")` — the constructor auto-orders them.

- `SpreadDefinition.compute_spread(lx, ly)` returns the **raw spread** (not z-scored).
  Use `compute_zscore(lx, ly)` for the normalised z-score.

- `RollingOLSConstructor._rolling_ols()` uses `ffill().bfill()` on gaps.
  This is intentional — avoids NaN propagation but means early observations
  inherit the first available beta.

- Agent `execute()` never raises. It always returns `AgentResult` with
  `status=FAILED` on error. Check `result.status` before consuming `result.output`.

- `DiscoveryPipeline.run()` respects `train_end` — universe eligibility, candidate
  generation, pair validation, spread fitting, and stability analysis all clip to
  `prices[prices.index <= train_end]`. Never pass full prices without setting `train_end`
  if you care about leakage.

- `StabilityAnalyzer` `min_stable_windows=0.60` means 60% of rolling ADF windows
  must pass. Reduce this threshold for shorter histories, not by lowering it system-wide.
  Construct `StabilityAnalyzer(min_stable_windows=0.40)` per-run.

- `DiscoveryScore.composite_score` is a *weighted* combination of 6 dimension scores
  (correlation, cointegration, stability, regime, economic, liquidity). A high composite
  does NOT guarantee all dimensions are strong — always inspect dimension scores before
  routing to portfolio.

- `ThresholdSet` invariants are enforced in `__post_init__`. Never construct a
  `ThresholdSet` with `stop_z < entry_z` or `exit_z >= entry_z` — they will be silently
  corrected. Check `ts.modifiers_applied` to see what was adjusted.

- `SignalQualityEngine.assess()` returns grade F for CRISIS, BROKEN, TRENDING regimes
  **regardless of conviction**. Do not route grade F signals to the portfolio layer.
  Check `quality.skip_recommended` before acting.

- `TradeLifecycleStateMachine.transition()` raises `ValueError` on invalid transitions.
  Always validate with `sm.state` before calling transition. Use `sm.can_enter()`,
  `sm.can_add()`, `sm.is_position_active()` as shorthand checks.

- `RegimeEngine.classify()` applies a safety floor: ML hooks cannot override BROKEN or
  CRISIS. If `RegimeFeatureSet.break_confidence > 0.80`, the result will always be BROKEN
  regardless of the ML hook's output.

- `build_regime_features()` clips `spread`, `prices_x`, `prices_y` to `as_of` internally.
  You must still pass the `as_of` parameter explicitly — defaulting to now is intentional
  only for live use; always set `as_of` for backtesting.

---

## How to Add a New Agent

1. Subclass `BaseAgent` in `agents/base.py` or a new module
2. Set `NAME`, `ALLOWED_TASK_TYPES`, `REQUIRED_PAYLOAD_KEYS`
3. Implement `_execute(task, audit) -> dict`
4. Register in `agents/registry.py` `_populate_default_registry()`
5. Add a test in `tests/test_agents.py`

---

## Migration & Deprecation — Contributor Safety

> **Read before contributing.** The following paths have been deprecated or removed.
> See `docs/migration/migration_ledger.md` for the full ledger with rationale.

### Forbidden Paths (do NOT use in new code)

| Old Path | Canonical Replacement | Reason |
|----------|----------------------|--------|
| `root/backtest.py:ExitReason` | `core/contracts.py:ExitReason` | Removed -- was an incompatible 10-value duplicate of the canonical 19-value enum |
| `secrets/` package | `secrets_mgmt/` package | Renamed -- old name shadowed Python stdlib `secrets` module, breaking numpy |
| `root.utils`, `root.data_loader`, `root.config_manager` | `common.utils`, `common.data_loader`, `common.config_manager` | Dead aliases removed -- use direct `common.*` imports |

### Deprecated Paths (still present, sunset pending)

| Path | Canonical Alternative | Sunset Condition |
|------|----------------------|------------------|
| `root/backtest.py:TradeSide` | `core/contracts.py:SignalDirection` | Migrate internal references, then remove |
| `root/trade_logic.py:TradeSide` | `core/contracts.py:SignalDirection` | Migrate internal references, then remove |
| `core/signals_engine.py` | `core/signal_pipeline.py` (ADR-006) | P1-PIPE COMPLETE: `use_signal_pipeline=True` is now the **default** in optimization_backtester.py. signals_engine.py still used by orchestrator/sql_store — deprecate after full migration |

### Walk-Forward Disambiguation

- **Calendar stability check:** `root/optimization_tab.py:_run_walkforward_for_params` — uses calendar
  segments with 63-day minimum floor. This is a **stability check, NOT true walk-forward.**
- **True walk-forward:** `research/walk_forward.py:WalkForwardHarness` — expanding-window walk-forward
  with purged splits, embargo, and correct train/test discipline.
- The optimization tab labels its output "walk-forward" in the UI but it is **calendar splitting only.**

---

## Discovery Layer — Critical Doctrine

**Correlation is a discovery primitive, NOT tradability proof.**
High correlation → candidate. Cointegration + stability + regime → tradable.
Never skip the validation pipeline for correlated pairs.

**False discovery is the primary risk at scale.**
With N=100 instruments, there are ~4,950 pairs. Even at 5% false-positive rate,
~247 pairs would pass by chance. Always report `candidate_yield_rate` and
`rejection_breakdown` in run artifacts. Be skeptical of pass rates > 20%.

**Residual mean reversion is the preferred alpha abstraction.**
OLS spread, rolling-OLS spread, or Kalman-filtered spread — the spread must
mean-revert. A cointegrated pair that doesn't produce a mean-reverting spread
in the current regime is not tradable today.

---

## How to Add a New Discovery Family

1. Create a class in `research/candidate_generator.py` with a `discover(prices, universe_snapshot, config) -> list[CandidatePair]` method
2. Add the enum value to `DiscoveryFamily` in `research/discovery_contracts.py`
3. Register the family in `CandidateGenerator.__init__()` family map
4. Add a test in `tests/test_discovery.py::TestCandidateGenerator`
5. Document the family's expected yield rate and false-positive characteristics

## How to Add a New Universe Definition

1. Add a `@staticmethod` method to `BuiltinUniverses` in `research/universe.py`
2. Register sector/industry metadata in `InstrumentRegistry._EQUITY_SECTOR_MAP`
   (or let `InstrumentRegistry.get_or_infer()` infer from prices)
3. Test via `UniverseBuilder.build()` with synthetic prices in `tests/test_discovery.py`

## How to Add a New Validation Test

1. Add the `RejectionReason` enum value in `research/discovery_contracts.py`
2. Add the check logic in `research/pair_validator.py` `_run_checks()`
3. Populate `report.rejection_reasons.append(RejectionReason.YOUR_NEW_REASON)`
4. Add a threshold constant to `ValidationThresholds` in `core/contracts.py` if tunable
5. Add a test in `tests/test_pair_validator.py`

## How to Add a New Spread Model

1. Add the enum value to `SpreadModel` in `core/contracts.py`
2. Create a constructor class in `research/spread_constructor.py`
3. Add a branch in `build_spread()` factory
4. Add a branch in `WalkForwardHarness._run_fold()` transform section
5. Add test in `tests/test_spread_constructor.py`

## Signal Layer — Critical Doctrine

**Signals are proposals, never automatic executors.**
`EntryIntent`, `ExitIntent`, and friends are rationale artifacts. The portfolio
layer must check `block_reasons`, apply its own risk limits, and approve before
execution. Never route directly from signal to order.

**Regime is first-class at signal time.**
Always run regime classification before computing thresholds or quality.
`ThresholdEngine.compute(regime=...)` uses regime to select the right threshold set.
`SignalQualityEngine.assess(regime=...)` hard-vetoes CRISIS/BROKEN/TRENDING.

**Exit logic is explicit.**
Use `ExitReason` enum values everywhere. Never use bare strings for exit reasons.
`ExitOversightAgent` and `ExitDiagnostics` both consume typed `ExitReason`.

## How to Add a New Signal Family (z-score variant, alternative spread, etc.)

1. Add the spread/signal variant to `core/signal_pipeline.py` (canonical per ADR-006) or create a new module
2. Ensure the new family computes a z-score and a conviction score [0,1]
3. Expose `regime` as a parameter — never hard-code thresholds inside the family
4. Wire conviction through `SignalQualityEngine.assess(conviction=..., mr_score=..., regime=...)`
5. Emit `EntryIntent` / `ExitIntent` (from `core/intents.py`) — not raw dicts
6. Add test in `tests/test_signal_engine.py`

## How to Add a New Regime Classifier (ML-based)

1. Implement `RegimeClassifierHookProtocol` from `core/regime_engine.py`:
   - `classify(features: RegimeFeatureSet) -> tuple[RegimeLabel, float]`
   - `model_id: str` property
2. Train with `PurgedKFold` (from `core/ml_validation.py`) to prevent leakage
3. Register via `RegimeEngine(ml_hook=your_classifier)`
4. **Respect the safety floor**: never return MEAN_REVERTING when `break_confidence > 0.80`
5. The `RegimeEngine` enforces the safety floor automatically — your classifier output
   is overridden for BROKEN and CRISIS regardless

## How to Add a New Threshold Scheme

1. Add a `RegimeThresholds` entry to `ThresholdConfig.regime_thresholds` for the new regime/variant
2. Or subclass `ThresholdConfig` with custom `base_entry_z`, `base_exit_z`, `base_stop_z`
3. Pass the config to `ThresholdEngine.from_config(config)`
4. Enforce: `stop_z ≥ entry_z + 1.0` and `exit_z ≤ entry_z - 0.5` (or let `ThresholdSet` fix it)
5. Add test in `tests/test_signal_engine.py::TestThresholdEngine`

## How to Add a New Lifecycle Rule (new state or transition)

1. Add the state value to `TradeLifecycleState` in `core/contracts.py`
2. Add a `StatePolicy` entry in `core/lifecycle.py` `_POLICIES` dict
3. Add valid transitions in `_VALID_TRANSITIONS`: `(FROM_STATE, TRIGGER) → TO_STATE`
4. Define a new `TRIGGER_*` string constant in `core/lifecycle.py`
5. Update `TradeLifecycleStateMachine` helpers (`can_enter`, `can_add`, `is_position_active`)
   if the new state changes those semantics
6. Add test in `tests/test_signal_engine.py::TestTradeLifecycleStateMachine`

## Portfolio Layer — Critical Doctrine

**A good pair is not automatically a good portfolio position.**
High statistical quality → candidate for funding. Composite ranking + constraint
checks → funded. Never route an EntryIntent directly to execution.

**Capital is scarce; signals compete.**
Every EntryIntent earns a composite score across 7 dimensions.
Capital flows to the highest-ranked fundable opportunities. Unfunded
opportunities are recorded with full rationale — not silently dropped.

**UNKNOWN cluster/sector skips concentration enforcement.**
If you don't pass `cluster_map` or `sector_map` to `run_cycle()`, the sector
and cluster concentration constraints are NOT enforced. Always pass metadata
in production — without it, crowding risk is invisible.

**EntryIntent uses `confidence` (not `conviction`).**
`BaseIntent.confidence` is the signal engine's conviction [0,1].
The portfolio layer's `RankedOpportunity.conviction` is the same value after
being extracted by the ranker. Don't confuse the two field names.

**Heat level is one-way downward automatically.**
`DrawdownManager.update()` can only worsen the heat level. To improve it,
call `DrawdownManager.attempt_recovery()` explicitly (at start of each cycle).

**Kill-switch only escalates automatically; recovery requires explicit reset.**
`KillSwitchManager.check()` only moves to a higher mode. To reset, call
`acknowledge()` then `reset()`. Use `reset(force=True)` to bypass the
acknowledgment requirement in automated testing.

## How to Add a New Portfolio Constraint

1. Add the enum value to `ConstraintType` in `portfolio/contracts.py`
2. Add the check logic in `PortfolioAllocator._check_constraints()`
3. Decide: severity "HARD" (blocks funding) or "SOFT" (warns only)
4. If tunable, add threshold to `ExposureConfig` or `AllocatorConfig`
5. Add test in `tests/test_portfolio.py::TestPortfolioAllocator`

## How to Add a New Ranking Dimension

1. Add field to `RankedOpportunity` in `portfolio/contracts.py`
2. Add weight to `RankingWeights` (adjust others so weights sum to 1.0)
3. Compute score in `OpportunityRanker.rank()` loop
4. Update composite formula in `rank()` and `_apply_diversification_scores()`
5. Add test in `tests/test_portfolio.py::TestOpportunityRanker`

## How to Add a New Sleeve

1. At startup: `capital_mgr.add_sleeve(SleeveDef("name", max_capital_fraction=0.20))`
2. Set `allowed_regimes`, `blocked_regimes`, `min_quality_grade` as needed
3. Modify `OpportunityRanker._suggest_sleeve()` to route pairs to the new sleeve
4. The sleeve-blocking constraint in `_check_constraints()` will enforce it automatically

---

## ML Platform (`ml/`)

> **Integration Status:** The ML platform infrastructure is complete and tested in isolation.
> As of 2026-03-31, no models have been trained and promoted to CHAMPION status, and
> `ModelScorer` is not called from the live signal generation path. All inference falls
> back to neutral probability (0.5). See `docs/remediation/remediation_ledger.md:P1-ML`.

Full documentation: `docs/ml_architecture.md`

### ML Platform Directory

```
ml/
├── contracts.py          # All ML domain types (source of truth for ml/ package)
├── features/             # PointInTimeFeatureBuilder + 61 FeatureDefinitions
├── labels/               # LabelBuilder + 26 LabelDefinitions
├── datasets/             # TemporalSplitter, LeakageAuditor, MLDatasetBuilder
├── models/               # MLModel base + MetaLabelModel, RegimeClassificationModel, BreakDetectionModel
├── evaluation/           # IC, AUC, Brier, meta_label_utility, walk-forward IC
├── registry/             # MLModelRegistry — champion/challenger, JSON persistence
├── inference/            # ModelScorer — TTL cache, tiered fallback, never raises
├── monitoring/           # FeatureDriftMonitor (PSI), ModelHealthMonitor
├── governance/           # GovernanceEngine — promotion criteria, usage contracts
└── explainability/       # FeatureLineageTracker, compute_feature_importance()
```

### ML Platform — Critical Doctrine

**ML is an overlay, not the foundation.**
Every ML decision has a deterministic rule-based fallback. If a model is stale,
drifted, or missing features, the system falls back to parametric rules — it never
fails silently.

**Point-in-time correctness is non-negotiable.**
All feature computation in `PointInTimeFeatureBuilder` clips data to `as_of` as
the first operation. Never pass full-history data without setting `as_of` in backtest.

**`ModelScorer` is the only correct inference path.**
Never call `model.predict()` directly at runtime. Use `ModelScorer.score()` — it
handles TTL caching, fallback order (CHAMPION → CHALLENGER → CANDIDATE → neutral),
and never raises exceptions.

**Always check `fallback_triggered`.**
`InferenceResult.fallback_triggered = True` means no model was available and a neutral
probability (0.5) was returned. Callers must inspect this flag before acting on the result.

**ML never overrides hard risk rules.**
`MLUsageContract.may_override_risk_limit` is always `False`. Kill-switch, drawdown
manager, and lifecycle state machine hard vetoes are never bypassed by ML output.

**Labels use future data by design.**
`LabelBuilder` methods use `shift(-horizon)`. This is correct — labels should reflect
future outcomes. Leakage prevention applies to *features*, not labels. All `LabelBuilder`
methods contain explicit `# LEAKAGE NOTE:` comments explaining why the forward look is intentional.

**Champion promotion requires governance approval.**
Never promote a model to CHAMPION directly via `registry.promote()` without first
passing `GovernanceEngine.check_promotion_criteria()`. Metrics required: IC ≥ 0.05,
AUC-ROC ≥ 0.55, Brier ≤ 0.25 (higher thresholds for regime/break models).

### ML Platform Gotchas

- `MLModel.fit()` raises `ValueError` if X contains timestamps after `train_end`. This
  is a hard leakage guard — not a soft warning. Set `train_end` correctly.

- `RegimeClassificationModel.classify()` applies a safety floor: if
  `RegimeFeatureSet.break_confidence > 0.80`, the model's output is overridden to BROKEN.
  The model cannot predict MEAN_REVERTING when the rule-based break detector is highly confident.

- `TemporalSplitter` purge + embargo are both required. Purge removes label-horizon overlap
  at the training boundary; embargo prevents near-boundary regime leakage in test. Use both.

- `LeakageAuditor` CRITICAL findings block training. Do not suppress them. They indicate
  a structural problem in feature/label alignment that must be fixed before training.

- `MLModelRegistry` auto-demotes the previous CHAMPION to RETIRED when a new CHAMPION
  is promoted. It does not delete the old model — it is always recoverable.

- `FeatureDriftMonitor.fit_reference()` should be called on out-of-time holdout data,
  not training data. Using training data as the reference defeats the purpose of drift detection.

- `CalibratedModelWrapper.calibrate()` must be called on held-out calibration data,
  never on training data. Calibration on training data inflates confidence scores.

### How to Add a New ML Task Family

1. Add `MLTaskFamily.YOUR_TASK` to `ml/contracts.py`
2. Define a `FeatureGroup` in `ml/features/definitions.py` → add to `FEATURE_GROUPS`
3. Define label(s) in `ml/labels/definitions.py` → add to `LABEL_REGISTRY`
4. Add label builder method in `ml/labels/builder.py` with explicit `# LEAKAGE NOTE:`
5. Add promotion thresholds in `ml/governance/policies.py` `PROMOTION_CRITERIA` dict
6. Create model class using `build_model()` factory or subclass `MLModel`
7. Wire protocol hook in the consuming core/portfolio module
8. Add tests covering: leakage check, temporal split correctness, inference fallback

### How to Add a New Feature

1. Create a `FeatureDefinition` in `ml/features/definitions.py` with correct `entity_scope`,
   `category`, `required_inputs`, and `lookback_days`
2. Add the name to `FEATURE_REGISTRY` dict
3. Add compute logic in `PointInTimeFeatureBuilder` — first line must clip to `as_of`
4. Add the feature to the appropriate `FEATURE_GROUPS` entry
5. If used in a canonical model, add to the canonical list in `ml/models/base.py`
   (e.g., `META_LABEL_FEATURES`, `REGIME_FEATURES`)

### How to Add a New Label

1. Create a `LabelDefinition` in `ml/labels/definitions.py` with `label_family`,
   `horizon_days`, `leakage_risks`, and `valid_use_cases`
2. Add to `LABEL_REGISTRY` dict
3. Add builder method in `ml/labels/builder.py` using `shift(-horizon)` for forward labels.
   Include `# LEAKAGE NOTE: this label uses data from [T, T+horizon] — correct by design`
4. Add test verifying temporal alignment (label index always < feature index by `horizon`)

### How to Promote a Model to Production

```python
from ml.registry.registry import get_ml_registry
from ml.governance.policies import GovernanceEngine
from ml.contracts import PromotionOutcome

registry = get_ml_registry()
engine = GovernanceEngine()

# Step 1: Register as CANDIDATE
registry.register(model)

# Step 2: Evaluate on out-of-time data
result = evaluator.evaluate(model, X_oot, y_oot)

# Step 3: Check governance criteria
decision = engine.check_promotion_criteria(training_artifact)
if not decision.approved:
    print("Failing criteria:", decision.failing_criteria)

# Step 4: Promote to CHALLENGER (shadow run)
registry.promote(model.model_id, PromotionOutcome.CHALLENGER)

# Step 5: After shadow evaluation, promote to CHAMPION
registry.promote(model.model_id, PromotionOutcome.CHAMPION)
# Previous CHAMPION is automatically demoted to RETIRED
```

---

## Agent Architecture

> **Integration Status:** Agents are registered in `AgentRegistry` and individually tested.
> As of 2026-03-31, no operational workflow dispatches agent tasks. `WorkflowEngine` is
> tested in isolation only. See `docs/remediation/remediation_ledger.md:P1-AGENTS`.

Full documentation: `docs/agent_architecture.md`

### How to Add a New Agent Safely

```
1. Choose the correct agent class module:
   - agents/research_agents.py:   discovery, validation, spread, regime, signal research
   - agents/ml_agents.py:         feature, label, model, meta-label, promotion
   - agents/monitoring_agents.py: health, drift, data integrity, incidents
   - agents/governance_agents.py: policy, approval, change impact, audit
   - agents/signal_agents.py:     signal classification, lifecycle, regime surveillance
   - agents/portfolio_agents.py:  allocation, exposure, drawdown, kill-switch

2. Subclass BaseAgent (from agents/base.py)

3. Set class-level constants:
   NAME = "unique_snake_case_name"          # must be unique across all agents
   ALLOWED_TASK_TYPES = ["task_type_a", ...]
   REQUIRED_PAYLOAD_KEYS = ["key1", ...]    # BaseAgent validates these on entry

4. Implement _execute(task, audit) -> dict
   - Call audit.log("message") for every significant decision
   - Never raise — let exceptions propagate to BaseAgent.execute() which wraps them
   - Always return a dict (never None; return {} on no-output tasks)
   - Wrap optional third-party imports in try/except ImportError with a fallback

5. Register in agents/registry.py _populate_default_registry()
   registry.register(YourNewAgent())

6. Add tests in tests/test_agent_architecture.py:
   - Test SUCCESS case with valid payload
   - Test FAILED case with missing required key
   - Verify audit.log() is called for key decisions
   - Verify result.output is never None

7. Document in docs/agent_architecture.md agent taxonomy table (Section 2)
   - Add row with: Agent, Module, NAME, Key Task Types, Action Boundary, Risk Class
```

### How to Add a New Workflow

```
1. Add a factory function in orchestration/engine.py (or a dedicated module)
   that returns a WorkflowDefinition.

2. Define WorkflowStep objects with all required fields:
   - step_id        : unique within this workflow
   - agent_name     : must match a registered BaseAgent.NAME
   - task_type      : must be in that agent's ALLOWED_TASK_TYPES
   - depends_on     : tuple of step_ids that must be COMPLETED first
   - timeout_seconds
   - retry_max
   - risk_class     : from RiskClass enum (orchestration/contracts.py)
   - requires_approval_before / requires_approval_after
   - on_failure     : "fail_workflow" | "skip" | "escalate" | "retry"

3. Define WorkflowTransition objects for every non-trivial routing decision:
   - from_step_id, to_step_id (None = terminal)
   - condition: "always" | "on_success" | "on_failure" | "on_approval" | "on_rejection"

4. Set entry_condition and termination_condition as clear human-readable strings.

5. Set appropriate environment_class and risk_class on the WorkflowDefinition.
   risk_class must equal the highest risk_class across all steps.

6. Export the factory from orchestration/__init__.py

7. Add a test that:
   (a) calls the factory and receives a non-None WorkflowDefinition
   (b) verifies step count matches the design
   (c) verifies every depends_on reference points to a valid step_id
   (d) verifies no step has risk_class > workflow.risk_class
```

### How to Add a New Approval Policy

```
1. In governance/engine.py _register_default_policies(), create and register
   a new GovernancePolicyVersion:
   - policy_id:         SCREAMING_SNAKE_CASE unique identifier
   - policy_name:       Human-readable name
   - version:           "1.0"
   - description:       What the policy enforces and why
   - rules:             tuple of human-readable rule statements
   - risk_class:        "ALL" or a specific RiskClass value string
   - environment_scope: tuple of environment strings where it applies
   - effective_from:    ISO date string
   - status:            PolicyStatus.ACTIVE

2. In GovernancePolicyEngine.check_policy(), add a branch for the new policy_id.
   The branch must:
   - Return PolicyCheckResult(passed=True) when the action complies
   - Return PolicyCheckResult(passed=False, severity=..., remediation_hint=...)
     when the action violates the policy
   - Never raise — exceptions become PolicyCheckResult failures

3. Test the policy:
   - Write a test that constructs an action dict that violates the policy
     and verifies passed=False with the expected severity
   - Write a test that constructs a compliant action dict and verifies passed=True

4. All policies must have: policy_id, policy_name, version, description,
   rules (list of human-readable strings), effective_from, and next_review dates.
   Policies without next_review dates are flagged by PolicyReviewAgent.
```

### How to Add a New Incident Workflow

```
1. In incidents/manager.py _register_default_runbooks(), add a RunbookReference:
   - runbook_id:              unique identifier
   - title:                   short description
   - applicable_severity:     tuple of IncidentSeverity.value strings (e.g. ("P0", "P1"))
   - applicable_components:   tuple of component identifiers this runbook covers
   - steps:                   ordered tuple of procedural steps (strings)
   - version:                 "1.0"
   - last_reviewed:           ISO date string

2. In IncidentManager.create_incident(), the keyword auto-linker scans the incident
   title and description for component names that match applicable_components.
   Add your component name to applicable_components so auto-linking works.

3. Use IncidentSeverity appropriately:
   - P0: complete system halt, trading suspended, immediate all-hands response
   - P1: major degradation, significant alpha or risk impact
   - P2: partial degradation, monitoring required
   - P3/P4: low-impact or informational

4. After resolution, call create_postmortem(incident_id, drafted_by=...) to
   capture learning. For P0/P1, this is called automatically by close_incident().

5. Every incident creation and update produces an AuditRecord:
   - INCIDENT_CREATED on create_incident()
   - INCIDENT_UPDATED on update_incident()
   Log these via incident_manager.log_audit() so they appear in the audit trail.
```

### How to Add a New Artifact Type

```
1. Add an enum value to agent_artifacts/contracts.py ArtifactType enum:
   YOUR_ARTIFACT_TYPE = "YOUR_ARTIFACT_TYPE"

2. Create a frozen dataclass for the artifact with required fields:
   @dataclass(frozen=True)
   class YourArtifact:
       artifact_id: str         # UUID
       artifact_type: ArtifactType  # = ArtifactType.YOUR_ARTIFACT_TYPE
       producing_agent: str
       timestamp: str           # ISO-8601 UTC
       schema_version: str = "1.0"
       # domain fields...
       confidence: float = 1.0  # if the artifact represents an assessment
       warnings: tuple[str, ...] = ()  # always include a warnings field

3. Export the new dataclass from agent_artifacts/__init__.py

4. Add test verifying:
   - Construction succeeds with valid inputs
   - Frozen behavior: attempting to assign a field after construction raises FrozenInstanceError
   - warnings field defaults to empty tuple (not None)
   - schema_version defaults to "1.0"
```

### How to Preserve Auditability

```
- Every significant decision inside _execute(): call audit.log("message")
  The AgentAuditLogger is injected into _execute() — use it, don't bypass it.

- Every workflow transition: WorkflowStepRun records agent_task_id and
  agent_result_status automatically. No manual action required.

- Every approval decision: ApprovalDecision links to ApprovalRequest via
  request_id, and ApprovalEngine logs AuditRecord(APPROVAL_DECISION) automatically.

- Every incident: call incident_manager.log_audit(AuditRecordType.INCIDENT_CREATED)
  from within create_incident(). Never create an incident without this audit record.

- Every emergency action: governance_engine.emergency_disable() and reenable_agent()
  both log AuditRecord(EMERGENCY_ACTION) automatically.

- Rule: if there is no AuditRecord for an action, the AUDIT_TRAIL_REQUIRED policy
  will flag it as a violation. The audit IS the safety proof. Never bypass it for
  performance or convenience.
```

### How to Avoid Uncontrolled Autonomy

```
- Agents return AgentResult. They never execute downstream actions directly.
  If an agent wants to trigger another agent, it returns a recommendation and lets
  the workflow engine or a human decide whether to act.

- Use ActionBoundary.INFORMATIONAL and ActionBoundary.RECOMMENDATION freely.
  These are the correct boundaries for the vast majority of analytical agents.

- Use ActionBoundary.CONTROLLED_OPERATIONAL only when the step is explicitly
  configured with risk_class and requires_approval fields in the WorkflowStep.
  The engine enforces this; do not call the operational action directly from _execute().

- ActionBoundary.SENSITIVE ALWAYS requires HUMAN_REQUIRED or DUAL_APPROVAL mode
  in the ApprovalEngine. There is no path to auto-approve a SENSITIVE action.
  Do not set ApprovalMode.AUTOMATIC for SENSITIVE actions even in RESEARCH env.

- Never increase an agent's permission set (AgentPermissionSet.allowed_capabilities)
  to bypass a governance check. If a capability requires approval, the correct fix
  is to wire the approval gate in the WorkflowStep, not to relax the permission.

- Emergency disable is always available for any misbehaving agent:
    get_governance_engine().emergency_disable("agent_name", reason="...")
  Use it. It is instant, logged, and does not require a process restart.

- max_delegation_depth=3 is enforced by the orchestration engine. Never bypass
  this by constructing AgentTask objects manually with a forged delegation_depth
  field. The engine reads this from the task payload and rejects violations.
```

---

## Production Operations Architecture

> **Integration Status:** Runtime infrastructure (RuntimeStateManager, ControlPlaneEngine,
> AlertEngine, ReconciliationEngine) is implemented and tested. As of 2026-03-31, no live
> or paper trading system exists. `is_safe_to_trade()` is called via safety_check in
> `run_portfolio_allocation_cycle()` (P1-SAFE COMPLETE). See `docs/remediation/remediation_ledger.md`.

Full documentation: `docs/production_architecture.md`

### How to Add a New Runtime Component Safely

```
1. Define the component's ServiceState lifecycle in runtime/contracts.py.
   - Add the component_id to any relevant EnvironmentSpec.requires_approval_for
     if activating/deactivating it in live requires approval.

2. Add heartbeat emission to the component's main loop:
   monitor = get_system_health_monitor()
   monitor.register_heartbeat(HeartbeatRecord(
       component="your_component",
       env=env,
       heartbeat_at=datetime.now(timezone.utc).isoformat(),
       sequence=seq,
       state="running",
       metadata={},
   ))
   Emit at least every heartbeat_gap_s seconds (default: 120).

3. Add a dedicated HealthStatus check in SystemHealthMonitor if the component
   needs non-heartbeat validation (e.g., connectivity test, table existence).
   - Follow the pattern in check_sql_store(): use a lazy import inside try/except.
   - Return HealthStatus with UNKNOWN severity on ImportError, not CRITICAL.

4. Add a control plane action type if operators need a non-standard control
   surface for this component (e.g., TRIGGER_DRAIN, FORCE_RECONCILE).
   - Add the value to ControlPlaneActionType in control_plane/contracts.py.
   - Add the dispatch branch in ControlPlaneEngine._dispatch().
   - Add a convenience method on ControlPlaneEngine.
   - Add a test in tests/test_production_ops.py.

5. Never let the component read RuntimeStateManager directly for is_safe_to_trade
   checks. Always call get_system_health_monitor().is_safe_to_trade(env) or
   get_runtime_state_manager().is_safe_to_trade(). Do not re-implement this logic.
```

### How to Add a New Alert Rule

```
1. Choose an AlertFamily from alerts/contracts.py that matches the domain.
   If none fits, add a new family value — but only if it represents a
   genuinely distinct alert domain.

2. Construct an AlertRule (frozen):
   rule = AlertRule(
       rule_id="YOUR_RULE_ID",              # SCREAMING_SNAKE_CASE
       name="Human-readable rule name",
       description="When this fires.",
       family=AlertFamily.YOUR_FAMILY,
       severity=AlertSeverity.ERROR,
       dedup_key="your_dedup_key",          # fires once per scope per window
       suppression_window_minutes=15.0,
       auto_resolve_minutes=None,           # or a number if self-resolving
       escalation_delay_minutes=30.0,
       requires_acknowledgement=True,
       runbook_url="https://runbooks.internal/your-rule",
       tags=frozenset({"your_tag"}),
       enabled=True,
   )

3. Register the rule in AlertEngine._register_default_rules():
   self.register_rule(rule)

4. Fire the rule in the relevant engine/check:
   alert_engine.fire(
       rule_id="YOUR_RULE_ID",
       source="your_module",
       scope="global",          # or a specific pair_id, strategy_id, etc.
       message="Description of what happened",
       context={"key": "value"},
   )

5. Add a test in tests/test_production_ops.py verifying:
   - The rule is present in list_rules()
   - fire() returns an AlertEvent (not None) on first call
   - fire() returns None on second call within suppression window
   - acknowledge() succeeds on the returned event
```

### How to Add a New Kill-Switch Scope

The kill switch is global — there is no per-strategy or per-model kill switch at the
runtime layer. "Scoped halts" are implemented through component-level ServiceState:

```
1. For a component-level halt (e.g., halt a single strategy):
   runtime_state_manager.update_component_state(strategy_id, ServiceState.HALTED)
   This causes is_safe_to_trade() to return False system-wide until resolved.

2. For exits-only mode on a component:
   control_plane.set_exits_only(reason="...", actor="...")
   This sets ThrottleLevel.EXITS_ONLY globally.

3. There is no partial kill switch — do not add one. The design is intentional.
   A partial halt that does not propagate to is_safe_to_trade() creates risk of
   inconsistent state across the signal, portfolio, and execution layers.

4. If you need component isolation, use ActivationStatus:
   runtime_state_manager.deactivate_strategy(strategy_id)
   The portfolio allocator checks ActivationStatus before routing any EntryIntent.
```

### How to Add a New Deployment Stage or Policy

```
1. Add the new stage value to DeploymentStage in deployment/contracts.py.

2. Add valid transitions FROM and TO the new stage in
   DeploymentEngine._VALID_TRANSITIONS (a set of (from_stage, to_stage) tuples).

3. If the new stage requires special handling (e.g., audit record, alert, freeze check):
   Add a branch in DeploymentEngine._on_stage_entry(release, new_stage).

4. If the stage involves live capital or activation:
   - Add it to the freeze check in transition_stage() so frozen environments
     block this transition.
   - Add it to the approval-required check if an approval_id should be mandatory.

5. Add a RollbackReason value if the new stage introduces a new failure mode.

6. Update docs/production_architecture.md Section 8 with the new stage in the
   lifecycle diagram.

7. Add tests in tests/test_production_ops.py verifying:
   - The new stage value exists in DeploymentStage
   - transition_stage() reaches it from the expected prior stage
   - Invalid predecessors raise ValueError or RuntimeError as appropriate
```

### How to Add a New Reconciliation Workflow

```
1. Add a new DiffType value to reconciliation/contracts.py if the workflow
   detects a new category of discrepancy.

2. Add a new method on ReconciliationEngine:
   def check_your_condition(self, ...) -> List[ReconcileDiffRecord]:
       ...

3. Call your new method from ReconciliationEngine.reconcile() and extend
   the ReconciliationReport with the resulting diffs:
   report.diffs.extend(self.check_your_condition(...))

4. If a critical diff should fire an alert automatically:
   if self._alert_engine and any critical diffs:
       self._alert_engine.fire(rule_id="RECONCILIATION_BREAK", ...)

5. Update is_clean() if the new diff type should block trading
   (i.e., contribute to critical_diffs count).

6. Add a test in tests/test_production_ops.py with:
   - A scenario that triggers the new diff type
   - Verification that is_clean() returns False when the diff is present
   - Verification that the diff type appears in report.diffs
```

### How to Add a New Runbook

```
1. Create a runbook entry in docs/production_architecture.md Section 14
   following the standard structure:
   ## Alert: <RULE_ID>
   ### When / ### Impact / ### Immediate Action / ### Escalation / ### Post-Incident

2. Set the runbook_url on the corresponding AlertRule to a stable URL or a
   local file path (e.g., "docs/production_architecture.md#alert-your-rule-id").

3. If the runbook covers a specific component, ensure that component name
   appears in your monitoring dashboard's component taxonomy so operators
   can navigate from alert to runbook without context switching.

4. Review the runbook quarterly. Update last_reviewed on the corresponding
   IncidentRunbook object (governance/contracts.py) after every review.
```

### How to Preserve Environment Separation

```
- NEVER hard-code environment names ("live", "paper") in domain logic.
  Domain modules (core/, research/, portfolio/) are environment-agnostic.
  Environment-specific behavior lives only in runtime/ and control_plane/.

- NEVER call get_runtime_state_manager() or get_control_plane() from inside
  core/, research/, portfolio/, ml/, or agents/ modules.
  These are production-layer singletons; domain logic must not depend on them.

- NEVER use RuntimeMode.LIVE as a conditional in signal or portfolio code.
  The correct pattern is: check is_safe_to_trade() and ThrottleLevel; never
  branch on the mode string.

- The environment spec (EnvironmentSpec) is the single source of truth for
  what is permitted in each environment. Do not duplicate these rules in
  domain modules.

- Secrets that are specific to an environment should use separate
  SecretReference objects with different key_path values (e.g., different
  env var names for paper vs live broker credentials). Never share credentials
  across environments.
```

### How to Avoid Hidden Live Behavior Changes

```
- Every change to RuntimeStateManager state must go through ControlPlaneEngine.
  Direct calls to RuntimeStateManager methods are only acceptable in tests
  or in the control plane itself.

- Never activate a strategy, model, or agent by mutating RuntimeStateManager
  from a background thread or scheduled job. All state changes must be
  operator-initiated through ControlPlaneEngine.execute_action().

- deployed != activated. Do not transition a release to ACTIVATED in an
  automated pipeline without an explicit approval gate. Even if the canary
  metrics look good, the final ACTIVATED transition requires an approval_id.

- Freeze windows exist for a reason. Never bypass is_frozen() with a direct
  stage transition. If a freeze is blocking a critical hotfix, an operator
  must explicitly unfreeze via set_freeze(env, False, actor, reason) and
  that action is logged.

- RuntimeOverride.is_emergency = True is for genuine emergencies only.
  Using it to bypass approval for routine changes is an audit violation.
  Every emergency override should be followed by a post-incident review.

- Throttle level changes and kill switch operations propagate immediately
  to all callers of is_safe_to_trade(). There is no delayed propagation.
  Operators must be aware that these changes are system-wide and instant.
```

---

## Governance, Compliance & Audit Layer

> **Integration Status:** Governance infrastructure (PolicyRegistry, GovernancePolicyEngine,
> AuditChainRegistry, SurveillanceEngine) is implemented and tested. As of 2026-03-31,
> no signal, allocation, or execution decision is gated by any governance check. Audit chains
> are empty for all operational decisions. See `docs/remediation/remediation_ledger.md:P1-GOV`.

Full documentation: `docs/governance_architecture.md`

### Package Directory

```
audit/          — AuditChain, hash-linked entries, AuditChainRegistry
evidence/       — EvidenceBundleBuilder, EvidenceItem, completeness checks
controls/       — ControlRegistry (15 defaults), ControlTestRecord, status lifecycle
policies/       — PolicyRegistry (versioned), PolicyConformanceReport
surveillance/   — SurveillanceEngine (12 rules), SurveillanceEvent, SurveillanceCase
exceptions_mgmt/— ExceptionEngine, ExceptionWaiver, CompensatingControl, expiry
attestations/   — AttestationEngine, recurring cadence, recertification
operating_model/— AccessControlManager (RBAC), SoD rules, RACI
reporting/      — GovernanceDashboard, GovernanceDashboardSummary (GREEN/AMBER/RED)
retention/      — RetentionManager (9 policies), LegalHold, deletion eligibility
```

### Governance Critical Doctrine

**Audit chains are append-only and hash-linked.**
Never modify an existing AuditChainEntry. Every entry carries a `prev_entry_id` and
`payload_hash`. Use `AuditChain.validate()` to detect tampering or missing links.

**Evidence must satisfy requirements before promotion.**
`EvidenceBundleBuilder.check_completeness()` returns `is_complete=False` if any
mandatory requirement is unsatisfied or any item is expired. Gate promotions on this check.

**Controls have a typed status lifecycle.**
ACTIVE → FAILED (on test failure) → ACTIVE (on recovery pass). DEGRADED is the intermediate
state for PARTIAL test results. `get_critical_failures()` surfaces single-points-of-failure.

**SoD violations are detected immediately on grant_role().**
If granting a role creates a conflict, the violation is recorded instantly. Run
`check_sod()` before live activation to ensure no outstanding violations.

**Attestations are the operating heartbeat.**
Recurring attestations (cadence_days > 0) auto-schedule the next request on completion.
`mark_overdue()` must be called regularly (e.g. daily) to surface missed attestations.

**Exception waivers have explicit expiry.**
`expire_waivers()` must be called to deactivate expired waivers. Use `has_active_waiver()`
before routing around a control or policy rule.

**Governance dashboard is read-only.**
`GovernanceDashboard.generate_summary()` never mutates subsystem state. GREEN requires
zero critical failures, no critical surveillance events, and no critical SoD violations.

### How to Add a New Control

1. Create a `ControlDefinition` with a unique `CTRL-XXX-NNN` ID in `controls/registry.py`
2. Add it to `_populate_defaults()` with appropriate domain, type, frequency, and criticality
3. Assign an owner via `assign_owner()` at startup
4. Write a test in `tests/test_governance.py` Section C
5. Add to `docs/governance_architecture.md` controls table

### How to Add a New Surveillance Rule

1. Create a `SurveillanceRule` with a `SURV-XX-NNN` ID
2. Add to `_populate_default_rules()` in `surveillance/engine.py`
3. Set `auto_escalate=True` for critical families (REGIME_BREACH, SPREAD_ANOMALY, DATA_INTEGRITY)
4. Handle threshold direction: for "lower is bad" metrics, add rule_id to `lower_is_bad_rules`
5. Test with `eng.detect(rule_id, entity_type, entity_id, metric_value)`

### How to Add a New Policy

1. Add rules as `PolicyRule` objects with unique `POL-XXX-RN` IDs
2. Call `policy_registry.register_policy(policy_id, rules, authored_by, change_summary)`
3. For HARD_LIMIT/PROHIBITED rules: set `automated_enforcement=True` — these are blocking
4. For SOFT_LIMIT/NOTIFY_REQUIRED: set `automated_enforcement=False` — advisory only
5. Inject violations in evaluation via context dict: `{f"rule_{rule_id}_passed": False}`

### How to Run the Governance Dashboard

```python
from reporting.dashboard import get_governance_dashboard

dashboard = get_governance_dashboard()
summary = dashboard.generate_summary()

print(f"Health: {summary.governance_health}")
print(f"Critical failures: {summary.critical_failures}")
print(f"Open surveillance: {summary.open_surveillance_events}")
print(f"Overdue attestations: {summary.overdue_attestations}")

if summary.governance_health == "RED":
    for concern in summary.top_concerns:
        print(f"  {concern}")
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benda931)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/benda931)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
