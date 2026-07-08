## idimikang-main

> Project notes for AI agents working in this repo.

# AGENTS.md — Idim Ikang

Project notes for AI agents working in this repo.

## Environment

- The freqtrade engine lives in `strategies/` (it IS the freqtrade source, imported as `freqtrade.*`).
- Runtime Python is **Windows Python 3.14** (`C:\Python314\python.exe`), user site-packages at `C:/Users/idona/AppData/Roaming/Python/Python314/site-packages`.
- pandas / numpy / scipy / scikit-learn are installed there.
- The full `freqtrade` stack cannot be imported on Windows Python 3.14 because the installed `ccxt` uses a metaclass incompatible with 3.14 (`Metaclasses with custom tp_new are not supported`). Syntax-check engine files with `python -c "import ast; ast.parse(open(f).read())"` rather than importing them on this host.
- **pwb-toolbox was removed**: the `Stocks-Quarterly-IncomeStatement` equities-fundamentals dataset is a category error for a crypto perpetual engine. Do not re-add it.

## Test / verify commands

```bash
python -m pytest tests/ -q                              # all tests (36)
python -m pytest tests/test_afml_toolkit.py -q          # AFML toolkit + trial registry
# AFML calibration against the 199 EMITTED sealed signals (must PASS pre-deploy):
python quant_core/afml_calibration.py
# syntax check a freqtrade engine file without importing it:
python -c "import ast; ast.parse(open('strategies/optimize/backtesting.py', encoding='utf-8').read())"
```

## Pre-deploy gates (must pass before AFML touches the live decision path)

1. **Math calibration**: `python quant_core/afml_calibration.py` must print `CALIBRATION: PASS`. This replays the 199 EMITTED signals through the corrector and checks (a) the OFF path is inert — sealed numbers unchanged when flags are disabled, and (b) the ON path's deflated Sharpe + PBO match independent hand-calculation. Last run: PASS.
2. **Trial count N is set**: the DSR correction requires the true cumulative N across the whole campaign. Before deploying, set it via one of:
   - `TrialRegistry.set_n(312)` in code (the known total across all passes),
   - `config["afml_n_trials"] = 312`,
   - or `IDIM_AFML_N_TRIALS=312` env var.
   If N is unset, the DSR refuses to deflate (raises `TrialCountUnknown` in strict mode, or warns + returns 1 in non-strict reporting mode). **Never** let N silently default to 1 — that is zero deflation, the exact failure mode this layer prevents.
3. **Engine-integration calibration (gate #3)**: `python quant_core/afml_gate3_engine_integration.py --config <config.json> --timerange <range>` must print `GATE #3: PASS`. This runs a real backtest with AFML flags off (confirms `results["afml"]` absent — inert wiring against the real engine, not the ledger stand-in) and on (confirms `afml` diagnostics surface without error — proves `_afml_postprocess` reads freqtrade's backtest output structure correctly). **Requires WSL where freqtrade/ccxt imports.** A `--smoke-test` mode is available that just confirms flags default OFF without a full backtest. Until this gate passes, the AFML layer must not touch live candidate decisions — the math calibration (gate #1) does not cover engine integration.

### Defect found and fixed during audit

`backtest_is_oos_split` originally defaulted to **0.3** (not 0.0), which meant the IS/OOS diagnostic ran on **every** backtest by default, computing a DSR with `n_trials=1` (registry empty, `strict=False` → 1). This is exactly the "side effect even when disabled" + "deflation theater" failure modes combined — every backtest got a meaningless DSR that looked rigorous. Fixed: default changed to 0.0 (disabled). The calibration script's OFF-path check masked this because it explicitly set `backtest_is_oos_split: 0.0`; the real engine default was different. This is why gate #3 (real engine, not the calibration stand-in) is load-bearing.

## AFML toolkit (`quant_core/afml/`)

Implements the backtesting-bias remedies from Lopez de Prado's *Advances in Financial Machine Learning*:

- `purged_k_fold.py` — `PurgedKFold`, `purged_train_test_split` (purge + embargo).
- `cpcv.py` — `CPCV`, `combinatorial_purged_cross_validation` (multiple backtest paths).
- `deflated_sharpe.py` — `deflated_sharpe_ratio`, `expected_max_sharpe` (selection-bias correction).
- `pbo.py` — `probability_of_backtest_overfitting` (PBO from CPCV paths).
- `walk_forward.py` — `WalkForwardSplit`, `walk_forward_windows` (walk-forward + embargo).
- `transaction_costs.py` — `TransactionCostModel`, `apply_transaction_costs` (spread + commission + slippage + impact).
- `diagnostics.py` — `backtest_report`, `is_oos_gap`, `sharpe_decay` (IS/OOS gap + verdict).
- `trial_registry.py` — `TrialRegistry`, `resolve_n_trials` (cumulative campaign-wide N for DSR; refuses to silently default).
- `quant_core/afml_guards.py` — `audit_lookahead`, `selection_bias_guard` (look-ahead + selection-bias guards for the signal engine).
- `quant_core/afml_calibration.py` — calibration harness mirroring `forward_adjudicator.backfill_calibration`.

All AFML imports in engine code are wrapped in `try/except ImportError` so a missing `quant_core` never breaks an engine.

## DSR trial count (N) — the quiet-bias-theater risk

The Deflated Sharpe Ratio's correction depends on N = the **total** number of strategy configurations tried across the entire campaign (every hyperopt eval, every regime variant, every exit policy, across every run) — NOT the number of trials in a single optimization pass. Using `config["epochs"]` under-counts by an order of magnitude and makes the deflation look rigorous while letting overfit candidates through. An N of 40 when the true campaign was 400 deflates as confidently and as wrongly as N=1 — it just hides better.

`TrialRegistry` (`quant_core/afml/trial_registry.py`) is the single source of truth for N:
- All three DSR call sites (hyperopt `afml_loss_helpers._n_trials`, backtesting `_afml_postprocess`, phase2 `_resolve_afml_n_trials`) read from it via `resolve_n_trials()`.
- It accumulates across runs (persistent JSON at `~/.idim_afml_trial_registry.json`, override via `IDIM_AFML_TRIAL_REGISTRY`).
- When N is unknown, `resolve_n_trials(strict=True)` raises `TrialCountUnknown` — it does NOT silently return 1.

### Trial-registration audit (which paths count toward N)

All registration is **opt-in** via `afml_count_trials: true` (config) or `IDIM_AFML_COUNT_TRIALS=1` (env). When off (the default), no path registers, the registry stays empty, and `resolve_n_trials(strict=True)` raises — the safe failure. When on, the following paths register:

| Path | Wired? | Register call | Notes |
|------|--------|---------------|-------|
| Hyperopt epochs | YES | `afml_loss_helpers._n_trials` → `get_registry().register(1, note="hyperopt_epoch")` | One per loss evaluation. Hyperopt calls `backtest()` directly, not `backtest_one_strategy`, so no double-count with the standalone path. |
| Standalone `freqtrade backtesting` | YES | `backtest_one_strategy` → `get_registry().register(1, note="standalone_backtest:{name}")` | One per strategy in `--strategy-list`. |
| Phase2 `phase2_validation.py` run() | YES | `get_registry().register(1, note="phase2_validation_run")` | Gated by `IDIM_AFML_COUNT_TRIALS`. |
| Phase2 `phase2_validation_simple.py` run() | YES | `get_registry().register(1, note="phase2_validation_simple_run")` | Gated by `IDIM_AFML_COUNT_TRIALS`. |
| Phase2 `phase2_validation_exact.py` main() | YES | `get_registry().register(1, note="phase2_validation_exact_run")` | Gated by `IDIM_AFML_COUNT_TRIALS`. |
| Phase2 `phase2_runner.py` run() | YES | `get_registry().register(1, note="phase2_runner_run")` | Gated by `IDIM_AFML_COUNT_TRIALS`. |
| Walk-forward windows | NO (intentional) | — | Walk-forward windows are evaluation windows of ONE strategy, not separate candidate configs. Registering them would over-count N. |
| Scanner live signals | NO (intentional) | — | Live signal generation is deployment, not candidate evaluation. |
| Ad-hoc hand-run scripts | NO (irreducible gap) | — | One-off Python scripts that call `backtest()` directly bypass all entry points. **Mitigation**: use `TrialRegistry.set_n(known_total)` to record the true count out-of-band, or `reset()` between campaigns. If you run ad-hoc tests and don't `set_n()`, N will be partial and the DSR will be wrong — confidently wrong, because the registry can't raise on trials it never heard about. |

**Known limitation**: the registry counts *evaluations*, not *unique configs*. Running the same config 5 times for debugging registers 5 trials, not 1. When the true unique-config count is known, use `set_n()` to override the accumulated count with the exact number. `reset()` between campaigns to avoid cross-campaign contamination.

## Engine wiring (config flags)

### FreqAI (`strategies/freqai/data_kitchen.py`)
Set `freqai.data_split_parameters.method = "purged_kfold"` (plus optional `embargo_pct`, default 0.01, and `freqai.feature_parameters.label_period_candles`) to use purged train/test split with embargo + label-horizon purging instead of sklearn's `train_test_split`. Falls back to standard split otherwise.

### Hyperopt (`strategies/optimize/hyperopt_loss/`)
- New loss `DeflatedSharpeHyperOptLoss` (registered in `strategies/constants.py`): `--hyperopt-loss DeflatedSharpeHyperOptLoss` optimises the Deflated Sharpe Ratio (corrects for the cumulative N from `TrialRegistry`, not just `config["epochs"]`).
- `SharpeHyperOptLoss` now also surfaces `deflated_sharpe` / `oos_sharpe` / `n_trials` on `backtest_stats` for reporting (objective unchanged).
- Shared helpers in `afml_loss_helpers.py`: `deflated_sharpe_from_loss_args`, `attach_afml_diagnostics`.

### Backtesting (`strategies/optimize/backtesting.py`)
- `backtest_realistic_costs: true` + optional `backtest_transaction_costs: {spread_bps, commission_bps, slippage_bps, impact_eta, default_adv}` → subtracts realistic costs, adds `tx_cost_usd` / `profit_abs_net` columns and `afml.tx_cost_total_usd` / `afml.profit_total_net` diagnostics.
- `backtest_is_oos_split: 0.3` → splits the backtest into IS/OOS by close_date, computes `afml.is_oos` (Sharpe gap + Deflated Sharpe + verdict).
- `walk_forward: {train_days, test_days, embargo_days, step_days, retrain_every}` → `run_walk_forward()` runs rolling OOS windows and reports `oos_sharpe` + `sharpe_decay_pct` per strategy under `results["walk_forward"]`.
- Per-strategy `afml` diagnostics are merged onto `self.results["strategy"][name]` for reporting.
- N for DSR read from `resolve_n_trials(self.config, strict=False)` — never from `config["epochs"]` alone.

### quant_core (`quant_core/phase2_validation.py`)
- Adds an `afml_audit` section to the phase-2 results JSON: `lookahead` (audit_lookahead) + `selection_bias` (deflated Sharpe guard) + aggregate `status`.
- N for DSR resolved via `_resolve_afml_n_trials()` → `resolve_n_trials(strict=False)` (warns if unset, does not silently default).
- AFML verdict can only **downgrade** the overall verdict (PASS→WATCH or →FAIL), never upgrade.

---
> Source: [Mo-101/IdimIkang-main](https://github.com/Mo-101/IdimIkang-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
