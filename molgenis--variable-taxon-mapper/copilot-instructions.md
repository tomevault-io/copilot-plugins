## variable-taxon-mapper

> - `src/` hosts the core pipeline: embedding + taxonomy helpers (`embedding.py`, `taxonomy.py`), pruning logic under `pruning/`, orchestration in `pipeline/`, and the reviewer CLI in `src/error_review_cli.py`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/` hosts the core pipeline: embedding + taxonomy helpers (`embedding.py`, `taxonomy.py`), pruning logic under `pruning/`, orchestration in `pipeline/`, and the reviewer CLI in `src/error_review_cli.py`.
- Entry scripts (`main.py`, `predict.py`, `check_pruned_tree.py`, `summarize.py`) sit at the repo root; configurations live in `config.example.toml` (copy to `config.toml`), datasets and run artifacts go under `data/`, and long-form notes stay in `doc/`. Use `tests/` to mirror `src/` module names when adding coverage.

## Build, Test, and Development Commands
- `uv sync && source .venv/bin/activate` provisions the Python 3.11 env from `pyproject.toml`.
- `python -m main config.example.toml` runs the full evaluation loop (ingest → prune → LLM match → metrics).
- `python -m predict config.example.toml --output data/predictions.csv` emits predictions without evaluation.
- `python -m check_pruned_tree config.example.toml --limit 10000 --output data/Keyword_coverage.csv` regresses pruning coverage.
- `python -m src.error_review_cli --predictions data/predictions.csv --keywords data/Keywords.csv` opens the interactive error reviewer.
- Batch or cluster jobs can reuse `bash run_pipeline_lb.sh` or the flow in `doc/nibbler_cluster.md`.

## Coding Style & Naming Conventions
Stick to type-hinted Python, one responsibility per module, and centralized helpers in `src/utils.py`. Keep functions/variables snake_case, classes PascalCase, and config keys lower_snake_case to match the TOML schema. New heuristics (embedding mixes, pruning scores) need a short comment plus an update to `config.example.toml`. Keep docstrings action-oriented and prefer dataclasses for structs passed between `matching`, `pruning`, and `reporting` code.

## Security & Configuration Tips
Never commit proprietary CSVs or llama.cpp weights; keep them in ignored paths and point configs to local copies. Load secrets via env vars read by `config.py`, and restrict llama.cpp endpoints to localhost (or add auth) before sharing tunnels.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/molgenis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/molgenis)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
