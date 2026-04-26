## financial-research

> **FPPE (Financial Pattern Prediction Engine)** — K-nearest-neighbor historical analogue

# CLAUDE.md — FPPE Project Root
# This file is loaded every session. Keep it under 80 lines.
# Detailed protocols live in .claude/skills/ — Claude loads them on demand.

## Project

**FPPE (Financial Pattern Prediction Engine)** — K-nearest-neighbor historical analogue
matching on return fingerprints. Generates probabilistic BUY/SELL/HOLD signals.

## Commands
- `PYTHONUTF8=1 py -3.12 -m pytest tests/ -q -m "not slow"` — Run full suite (always run before committing)
- `venv\Scripts\activate` — Windows venv activation

## Codebases

- `pattern_engine/` — Phase 3Z production core: PatternMatcher, Platt calibration, contracts
  - `matcher.py`: 5-stage PatternMatcher (scale → search → filter → aggregate → calibrate)
  - `contracts/`: Pydantic schemas, BaseMatcher ABC, EngineState, SignalDirection/Source
  - `features.py`, `data.py`, `schema.py` — feature pipeline and data loading
  - `regime.py`: RegimeLabeler (SPY + VIX spike + yield curve); regime_filter=False in prod
  - `live.py`: LiveRunner (Phase 5): receives AllocationDecisions + exit_tickers, submits via OrderManager + BaseBroker DI
  - `contracts/matchers/hnsw_matcher.py`: save_index/load_index for disk persistence
  - Research pilots (behind flags): `sax_filter.py`, `wfa_reranker.py`, `ib_compression.py`, `conformal_hooks.py`
- `trading_system/` — Phase 3Z production layer: SharedState, StrategyEvaluator, risk overlays
  - `strategy_evaluator.py`: signal → position decision with risk overlays
  - `signal_adapter.py`: UnifiedSignal (Pydantic), KNN/DL adapters
  - `risk_overlays/`: fatigue accumulation, liquidity congestion
  - `position_sizer.py`: Half-Kelly with SizingConfig (Phase 2)
  - `risk_engine.py`: stateless orchestrator — compute_atr_pct, drawdown_brake_scalar, apply_risk_adjustments (Phase 3)
  - `portfolio_manager.py`: rank_signals, allocate_day — stateless PM (Phase 4, when created)
  - `portfolio_state.py`: PortfolioSnapshot, RankedSignal, PMRejection (Phase 4, when created)
  - `broker/`: BaseBroker ABC, Order/OrderResult schemas, MockBroker (Phase 5)
  - `order_manager.py`: OrderManager — AllocationDecision→Order lifecycle (Phase 5)
  - `reconciliation.py`: Position reconciliation vs broker (Phase 5)
  - `drift_monitor.py`: feature drift detection
- `research/` — pluggable ABCs + Phase C modules
  - `hnsw_distance.py`: HNSWIndex, 54.5× speedup, recall@50=0.9996 (SLE-47 ✓)
  - Enable: `EngineConfig(use_hnsw=True)` — default False (ball_tree unchanged)
- `archive/legacy_v1/` — Pre-Phase-3Z legacy code (archived, do not modify)
- `rebuild_phase_3z/` — Phase 3Z rebuild workspace (preserved for reference + parallel tests)
  - `artifacts/baselines/parity_snapshot.json` — frozen SLE-80-v1 snapshot

## Critical Rules

1. **Run tests first.** `PYTHONUTF8=1 py -3.12 -m pytest tests/ -q -m "not slow"` — 807 tests, all must pass.
2. **Numbers require provenance.** Any claimed metric must trace to walk-forward results
   or experiment logs. If it cannot be traced, it is fabricated. No exceptions.
3. **Do NOT modify `prepare.py` or this file** unless explicitly asked.
4. **assert → RuntimeError** for all public API guards. `assert` is stripped under `-O`.
5. **nn_jobs=1** always. Prevents Windows/Py3.12 joblib deadlock.
6. **3-strike rule:** If three consecutive attempts at the same fix fail, STOP. Log what
   was tried in the session log and escalate.

## Locked Settings (do not change without new experiment evidence)

Distance=Euclidean, Weighting=uniform, Features=VOL_NORM_COLS(8), Calibration=beta_abm,
cal_frac=0.76, max_distance=0.90, top_k=50, confidence_threshold=0.65,
regime=hold_spy_threshold+0.05, horizon=fwd_7d_up, stop_loss_atr_multiple=3.0
# max_distance=0.90, beta_abm: swept H5 (2026-04-02). Best at 52T VOL_NORM. Provenance: results/bss_fix_sweep_h5.tsv
# regime=hold_spy_threshold+0.05: H7 (2026-04-06). GATE MET: 3/6 positive folds, mean_BSS=+0.00033.
#   mode=hold: Bear rows (SPY ret_90d < +0.05) → base_rate prob (HOLD signal).
#   Bull mode only: KNN signal used when SPY 90d return > +5% (confirmed trend).
#   Provenance: results/bss_fix_sweep_h7.tsv. Caution: thin margin, aggressive threshold.
# stop_loss_atr_multiple: swept 2.0–4.0 on 2024 fold (2026-03-21). 3.0× won:
# Sharpe=1.53 (+32% vs 2.0×), MaxDD=5.7%, stops=28/171. Provenance: results/atr_sweep_results.tsv
# confidence_threshold=0.65 locked; use 0.55 in experiments on 585T (max Platt prob=0.6195).
# 52T beta_abm probability range: [0.50, 0.58] — below 0.65 threshold. Use production signals
#   (results/cached_signals_2024.csv, 585T Platt) for backtest work, NOT the 52T walk-forward.
# min_sector_lift=0.005 (recalibrated 2026-03-26; old 0.03 vetoed 100% on 585T).
# BSS root cause (2026-03-26): analogue pool dilution at 585T scale, NOT miscalibration.
#   Platt is helping (+0.023 BSS vs raw). Fix: tighten max_distance or same_sector_only
#   — locked settings, require new experiment evidence before changing.

## Key Design Docs (read before modifying related code)

- `docs/FPPE_TRADING_SYSTEM_DESIGN.md` v0.3 — Trading layer architecture
- `docs/campaigns/PHASE_3Z_CAMPAIGN.md` — Full Phase 3Z rebuild history (SLE-51–89)
- `docs/CANDLESTICK_CATEGORIZATION_DESIGN.md` v0.2 — Future Phase 6
- `docs/PHASE2_SYSTEM_DESIGN.md`, `docs/PHASE2_RISK_ENGINE.md` — Phase 2 spec + campaign (in docs/, NOT docs/campaigns/)
- Phase 4 plan: `docs/superpowers/plans/2026-04-06-phase4-portfolio-manager-plan.md`

## Current Phase

**Phase 4 Portfolio Manager — COMPLETE (2026-04-09)**
- Plan: `docs/superpowers/plans/2026-04-08-phase4-portfolio-manager-plan.md`
- Result: Sharpe=2.649, MaxDD=4.4%, max sector=15.0%, mean idle cash=28.0% (2024 fold).
  8/9 gates PASS; G6 (rejection diversity) sample-size-gated N/A at n=9.
- Commits on `phase4-portfolio-manager` branch: T4.0b → T4.0 → T4.0c → T4.1a → T4.1b → T4.2 → T4.3 → T4.4.
- Session log: `docs/session-logs/SESSION_2026-04-09_phase4-t4-1a-through-t4-2.md`

**Phase 5 Live Execution Plumbing — IN PROGRESS (2026-04-09)**
- Plan: `docs/superpowers/plans/2026-04-09-phase5-live-plumbing-plan.md`
- Spec: `docs/superpowers/specs/2026-04-09-phase5-live-plumbing-design.md`
- T5.1–T5.6 complete (BaseBroker, MockBroker, OrderManager, Reconciliation, LiveRunner, Gate Test)
- G1: 100/100 fills ✓, G2: 30-day recon ✓, G3: 0.18s ✓ — T5.7 (CLAUDE.md) pending

Phase 1–4 summary (for context, not active):
- H7 regime HOLD: mean_BSS=+0.00033, 3/6 positive folds. Thin margin.
- Phase 2: Half-Kelly, Sharpe=2.527, 5/6 folds positive Kelly. 2022-Bear Kelly=-0.504.
- Phase 3: Risk engine, Sharpe=2.659, MaxDD=4.3%. Fatigue OFF (SLE-75 saturates).
- Phase 4: PM filter, Sharpe=2.649 (regression −0.38%), MaxDD=4.4%. 9/278 rejected, all insufficient_capital.

## Session Protocol

Every session must:
1. Read this file (automatic in Claude Code)
2. Check for active campaign: `docs/campaigns/`
3. Before ending: update session log via `/session-handoff`

## Environment

- Windows 11, Ryzen 9 5900X (12 cores), 32GB RAM, Python 3.12
- venv: `C:\Users\Isaia\.claude\financial-research\venv`
- Activate: `venv\Scripts\activate`

## Key Result Files

- `results/backtest_trades.csv` — Phase 1 flat 5% sizing, 278 trades, 2024 fold
- `results/phase2_backtest_trades.csv` — Legacy Phase 2 ATR 10% sizing, 191 trades, 2024 fold
- `results/cached_signals_2024.csv` — 585T Platt signals, 13,104 rows, 159 BUY, conf [0.65–0.75]
- `results/phase2_walkforward.tsv` — Kelly sizing, 6-fold results
- `results/phase3_walkforward.tsv` — Risk engine integration, 2024 fold
- `results/phase3_gate_check.txt` — Phase 3 gate metrics (Sharpe=2.659, MaxDD=4.3%)

## Skills Available

Claude: check `.claude/skills/` for task-specific protocols. Key skills:
run-walkforward, debug-bss, add-ticker, add-feature-set, run-backtest,
phase2-risk-engine, session-handoff

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IsaiahKing16) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
