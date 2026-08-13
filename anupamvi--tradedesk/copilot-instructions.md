## tradedesk

> - Treat `codexdaily`, `codex daily`, and discussions of the daily Codex pipeline as **Codex Daily V2** unless the user explicitly asks for V1, `uwos`, historical replay, or another pipeline.

# Trading Desk Agent Instructions

## Codex Daily V2 Default

- Treat `codexdaily`, `codex daily`, and discussions of the daily Codex pipeline as **Codex Daily V2** unless the user explicitly asks for V1, `uwos`, historical replay, or another pipeline.
- V2 is the clean `codexuw` implementation: `python3 -m codexuw.daily`.
- Default V2 live-planning command for a dated UW folder:

```bash
python3 -m codexuw.daily \
  --base-dir /Users/anuppamvi/uw_root/tradedesk/YYYY-MM-DD \
  --out-dir /Users/anuppamvi/uw_root/tradedesk/out/codexdaily_v2_YYYY-MM-DD \
  --max-tickers 60 \
  --max-candidates 50 \
  --max-final-trades 8 \
  --risk-budget 15000 \
  --monthly-profit-target 10000 \
  --max-contracts-per-trade 20 \
  --minimum-expected-value-per-dollar-risk 0.01 \
  --risk-mandate target-growth \
  --index-income-mode primary \
  --portfolio-income-mode trading-sleeve-only
```

- V2 must still use Schwab live chains/pricing and Schwab portfolio state for execution decisions. Do not force trades when live edge, liquidity, catalyst, or risk controls do not support them.
- Core investment holdings are protected by default. Do not recommend covered calls on long-term holdings unless the user explicitly allows the ticker or asks for `--portfolio-income-mode existing-core-review`.

## Pattern Analysis V2 Default

- Treat `pattern analysis`, `run pattern analysis`, `pattern-analysis`, and plain `pattern` requests as **Pattern Analysis V2** unless the user explicitly asks for V1, the frozen baseline, or `uwos.options_pattern_pipeline_v1`.
- V2 is the hardened Pattern Analysis entrypoint: `python3 -m uwos.pattern_analysis_v2`.
- The compatibility module `python3 -m uwos.options_pattern_pipeline_v2` routes to the same V2 engine.
- Default V2 command for latest source-complete local UW data:

```bash
python3 -m uwos.pattern_analysis_v2 \
  --base-dir /Users/anuppamvi/uw_root/tradedesk \
  --as-of latest
```

- Default V2 dated output path is:

```text
/Users/anuppamvi/uw_root/tradedesk/out/pattern_analysis_v2/YYYY-MM-DD
```

- V2 emits ticket-first `AUTO_APPROVED`, `TRADE_REVIEW`, `AVOID`, and `NO_TRADE` decisions with decision boards, manifests, walk-forward performance, threshold sensitivity, calibration, shadow ledger, profitability audit, and runbook artifacts.
- No trade/order placement.

## Pattern Pipeline V1 Baseline

- `uwos/options_pattern_pipeline_v1_frozen_v1/` is the immutable backup copy of the Options Pattern Pipeline V1 baseline.
- Do not edit, delete, rename, reformat, regenerate, or bulk-update files under `uwos/options_pattern_pipeline_v1_frozen_v1/` unless the user explicitly asks to update the frozen V1 baseline.
- Future pattern-pipeline work should happen in `uwos/options_pattern_pipeline_v1/` or a newly named successor package such as `uwos/options_pattern_pipeline_v2/`.
- If a future change needs rollback to V1 behavior, restore from `uwos/options_pattern_pipeline_v1_frozen_v1/` instead of reconstructing the code manually.
- Before committing future pattern-pipeline changes, verify that `uwos/options_pattern_pipeline_v1_frozen_v1/` has no accidental diff.

---
> Source: [Anupamvi/tradedesk](https://github.com/Anupamvi/tradedesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
