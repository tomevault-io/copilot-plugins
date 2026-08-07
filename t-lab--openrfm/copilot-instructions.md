## openrfm

> This repository is a KumoRFM-2 reproduction scaffold. The main Python package lives in `implementation/kumorfm_repro/`, with model code in `model.py`, synthetic data in `data.py`, training in `train.py`, RelBench/file-backed adapters in `benchmark_adapter.py` and `relational_io.py`, suite runners in `relbench_suite.py` and `synthetic_suite.py`, and target comparison in `targets.py`. Documentation and status notes live in `implementation/README.md` and `progress_plan/reproduction_status.md`. The source paper is in `technical_report/`. Generated datasets, manifests, checkpoints, and comparisons are under `implementation/data/`, `implementation/runs/`, and `implementation/targets/`; treat these as artifacts, not source modules.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a KumoRFM-2 reproduction scaffold. The main Python package lives in `implementation/kumorfm_repro/`, with model code in `model.py`, synthetic data in `data.py`, training in `train.py`, RelBench/file-backed adapters in `benchmark_adapter.py` and `relational_io.py`, suite runners in `relbench_suite.py` and `synthetic_suite.py`, and target comparison in `targets.py`. Documentation and status notes live in `implementation/README.md` and `progress_plan/reproduction_status.md`. The source paper is in `technical_report/`. Generated datasets, manifests, checkpoints, and comparisons are under `implementation/data/`, `implementation/runs/`, and `implementation/targets/`; treat these as artifacts, not source modules.

## Build, Test, and Development Commands

Work from the implementation directory:

```bash
cd /home/user/work/KumoRFM/implementation
python -m compileall kumorfm_repro
```

Use `compileall` as the fast syntax/import smoke check. Run a small synthetic training smoke test with:

```bash
python -m kumorfm_repro.train --steps 3 --batch-size 2 --context-size 8 --rows-per-child 4 --d-model 32 --layers 1 --output-dir runs/single_smoke
```

For two-GPU DDP checks:

```bash
torchrun --standalone --nproc_per_node=2 -m kumorfm_repro.train --steps 4 --batch-size 4 --context-size 8 --rows-per-child 4 --d-model 32 --layers 1 --amp --output-dir runs/ddp_smoke
```

For RelBench presets, inspect available suites with `python -m kumorfm_repro.relbench_suite --list-presets`.
Use `benchmark_adapter train-model` and `relbench_suite` as fine-tuning smoke paths only. Paper evidence should come from synthetic pre-training plus `icl_eval`/`icl_suite`.

## Coding Style & Naming Conventions

Use Python 3 with 4-space indentation, type hints where they clarify interfaces, and descriptive snake_case names for functions, variables, and CLI flags. Keep modules CLI-friendly via `python -m kumorfm_repro.<module>`. Prefer structured manifests (`.json`) and parquet/csv table IO over ad hoc text formats. Keep comments short and reserved for non-obvious model, data, or distributed-training behavior.

## Testing Guidelines

There is no dedicated test directory yet. Validate changes with `python -m compileall kumorfm_repro`, then run the smallest relevant smoke path: synthetic training for model/data changes, `benchmark_adapter validate` or `batch-smoke` for relational IO changes, and `relbench_suite --dry-run` for orchestration. Preserve deterministic seeds and JSON manifests.

## Commit & Pull Request Guidelines

This working tree does not include local Git history, so no project-specific commit convention can be inferred. Use concise imperative commit messages, such as `Add RelBench coverage-aware export`. PRs should describe the change, list exact commands run, link produced manifests under `implementation/runs/` or `implementation/targets/`, and call out GPU, download, or coverage assumptions.

## Agent-Specific Instructions

Do not delete or overwrite existing run artifacts unless explicitly requested. Prefer adding new run directories for experiments. Keep `progress_plan/reproduction_status.md` aligned with meaningful new evidence, especially benchmark metrics, failures, and command lines.

---
> Source: [T-Lab/OpenRFM](https://github.com/T-Lab/OpenRFM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
