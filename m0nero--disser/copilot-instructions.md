## disser

> `bio/` contains the BIO segmentation pipeline: CLI dispatch in `bio/cli.py`, configs in `bio/configs/`, model code in `bio/core/`, training and preprocessing stages in `bio/pipeline/`, and IPN helpers in `bio/ipn/`. `msagcn/` holds isolated-sign classification code, with loaders in `msagcn/data/`, network components in `msagcn/models/`, and training logic in `msagcn/training/`. `coreml/exporter/` contains the CoreML export CLI. Use `scripts/` and `utils/` for one-off extraction, diagnostics, and reporting. `notebooks/` is exploratory; keep production logic in packages, not notebooks.

# Repository Guidelines

## Project Structure & Module Organization
`bio/` contains the BIO segmentation pipeline: CLI dispatch in `bio/cli.py`, configs in `bio/configs/`, model code in `bio/core/`, training and preprocessing stages in `bio/pipeline/`, and IPN helpers in `bio/ipn/`. `msagcn/` holds isolated-sign classification code, with loaders in `msagcn/data/`, network components in `msagcn/models/`, and training logic in `msagcn/training/`. `coreml/exporter/` contains the CoreML export CLI. Use `scripts/` and `utils/` for one-off extraction, diagnostics, and reporting. `notebooks/` is exploratory; keep production logic in packages, not notebooks.

## Build, Test, and Development Commands
`python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` creates a clean environment. `python -m bio -h` lists the BIO pipeline commands. `python -m msagcn.train --json datasets/skeletons --csv datasets/data/annotations.csv --out outputs/runs/agcn_run` starts AGCN training. `bash bio/tools/smoke_test.sh` runs the smallest end-to-end BIO sanity check. `python -m coreml.exporter --ckpt outputs/runs/best.ckpt --out outputs/coreml/STGCN.mlmodel` exports a trained checkpoint.

## Coding Style & Naming Conventions
Follow existing Python style: 4-space indentation, `snake_case` for modules, functions, variables, and CLI flags, and `CamelCase` for classes. Keep module entry points small and centered on `main()` plus `argparse`, matching `bio/cli.py` and the scripts under `scripts/` and `utils/`. Add type hints for public functions when practical; the current pipeline code already uses them heavily. Name JSON configs with lowercase underscores, for example `bio_default.json`.

## Testing Guidelines
There is no formal `pytest` suite or coverage threshold in this repository today. For pipeline changes, run `bash bio/tools/smoke_test.sh` or `python -m bio smoke-test ...` on a tiny dataset before opening a PR. For training or export changes, include the exact command you ran and verify outputs are written under `outputs/` or `runs/`. The files under `test/` and `test1/` are sample fixtures, not an automated test harness.

## Commit & Pull Request Guidelines
Recent commits use short, imperative, mostly lowercase subjects such as `change paths`, `refactor coreml`, and `extract skeleton data`. Keep commits scoped to one concern and mention the subsystem when useful, for example `bio: tighten smoke-test defaults`. PRs should summarize data or config assumptions, list validation commands, and include metrics, plots, or screenshots when model behavior changes. Do not commit `datasets/`, `outputs/`, `runs/`, or virtual environment contents.

## Paths & Artifacts
This repo is designed around local data and generated artifacts being ignored by git. Keep new paths configurable or repo-relative; avoid hardcoded absolute machine paths in code, configs, and scripts.

---
> Source: [M0nero/disser](https://github.com/M0nero/disser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
