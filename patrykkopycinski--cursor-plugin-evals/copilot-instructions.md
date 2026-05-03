## post-run-analysis

> >-


# Post-Run Analysis — Auto-Diagnose and Fix

When evaluation result files are created or updated, IMMEDIATELY analyze them and fix any issues found. Do NOT just report — implement fixes.

## What to Do

1. **Read the result file** and compute:
   - Overall pass rate
   - Per-suite and per-evaluator pass rates
   - Failure clustering (are multiple tests failing for the same reason?)
   - Cost analysis (which tests are expensive?)

2. **If pass rate < 80%**:
   - Identify the root cause for each failing test
   - If the test expectation is wrong (e.g., tool was renamed), fix the YAML
   - If the evaluator threshold is too strict for the current maturity, adjust it
   - If a tool genuinely fails, flag it as a real bug in the plugin

3. **If regressions detected** (compared to fingerprint baseline):
   - Identify which tests regressed and by how much
   - Check if a recent code change caused the regression
   - If the test expectation needs updating, update it

4. **Auto-fix these patterns**:
   - Wrong tool name in `expected.tools` → correct it
   - Overly strict threshold causing false failures → relax to 2 standard deviations from mean
   - Missing evaluator causing incomplete scoring → add the evaluator
   - Flaky test (passes sometimes, fails others) → increase `repetitions` to 3

5. **After fixes, re-run** the failing suite to validate:
   ```bash
   npx cursor-plugin-evals run --suite FAILING_SUITE --verbose
   ```

6. **If content was modified** (skill scripts, SKILL.md, reference docs — not just eval YAML),
   run a targeted content audit on the changed files before declaring done:
   - Check blast radius: sibling copies of modified shared modules, referencing docs
   - Classify findings by severity (CRITICAL/HIGH/MEDIUM/LOW/INFO)
   - Fix all HIGH+ issues, re-check, repeat until no HIGH/MEDIUM remain (max 5 passes)

## When to Stay Silent

- Don't re-analyze the same result file if no changes occurred

## When Pass Rate is High — Calibrate Thresholds

When pass rate is above 90%, **don't just say "all looks good"**. Analyze whether the
CI thresholds are properly calibrated:

1. **Compute headroom** for each CI gate:
   - `headroom = actual_score - threshold`
   - `headroom_pct = headroom / threshold * 100`

2. **Flag over-generous thresholds**:
   - Headroom > 20%: threshold is stale — **recommend bumping** to `actual - 5%`
   - Headroom > 10%: threshold could be tighter — recommend review
   - Headroom < 5%: well calibrated — leave alone

3. **Identify weakest tests** dragging the average down:
   - If suite average is 0.95 but one test scores 0.72, that test is the bottleneck
   - Recommend fixing the weak test rather than keeping thresholds low to accommodate it

4. **Check for always-perfect scores**:
   - If a test scores 1.0 every time across 3+ runs, it may be too easy
   - Consider increasing difficulty or adding more challenging variants

5. **Report threshold calibration suggestions**:
   ```
   ## Threshold Calibration Suggestions
   - score.avg: 0.80 → 0.89 (actual: 0.94, headroom was 17.5%)
   - tool-selection.avg: 0.80 → 0.92 (actual: 0.97, headroom was 21.3%)
   - Weakest test: esql-complex-query (0.72) — consider improving prompt
   ```

6. **Apply the suggestions autonomously** — update `plugin-eval.yaml` CI section,
   re-run `--ci` to verify the tighter thresholds pass, then commit.

---
> Source: [patrykkopycinski/cursor-plugin-evals](https://github.com/patrykkopycinski/cursor-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
