## yolo-bad-triangle

> Framework-first architecture guardrails


# Framework-First Architecture Guardrails

This repository is framework-first. Extend the existing runner, registries, and reporting modules rather than adding parallel execution or contract systems.

- Canonical CLI entrypoints are `scripts/run_unified.py` (single run) and `scripts/sweep_and_report.py` (matrix sweeps). The core orchestration lives in `src/lab/runners/run_experiment.py` (`UnifiedExperimentRunner`).
- Run artifacts are written under `outputs/framework_runs/<run_name>/` (`metrics.json`, `predictions.jsonl`, `run_summary.json`). Sweep reports go under `outputs/framework_reports/<sweep_id>/`.

## Hard Constraints

1. Do not add new registry systems if a registry already exists:
   - `src/lab/runners/run_experiment.py` (runner orchestration)
   - `src/lab/attacks/framework_registry.py`
   - `src/lab/defenses/framework_registry.py`
   - `src/lab/models/framework_registry.py`
2. Do not create new ad-hoc entrypoints that bypass `run_unified.py` / `sweep_and_report.py` for normal lab workflows.
3. Do not duplicate schema or contract definitions outside `schemas/v1/` and `src/lab/config/contracts.py`.
4. Keep output validation and artifact checks aligned with `scripts/ci/validate_outputs.py` and `src/lab/health_checks/` (required files and JSON schemas for framework runs).

## Preferred Change Pattern

- Reuse existing modules first, then refactor for extension points.
- Prefer centralizing derived metrics and report tables in `src/lab/eval/derived_metrics.py` and `src/lab/reporting/framework_comparison.py` rather than new one-off report builders.
- Add or adjust tests under `tests/` when behavior or contracts change.

## Quick Example

- Bad: add a second sweep driver that shells out differently and writes non-contract JSON.
- Good: extend `scripts/sweep_and_report.py` or `src/lab/reporting/framework_comparison.py` and keep output shapes consistent with `schemas/v1/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cmaddock99) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
