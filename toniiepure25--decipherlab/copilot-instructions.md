## decipherlab

> - `src/decipherlab/ingest`: manifest readers and dataset adapters

# DecipherLab Working Guide

## Repo Map

- `src/decipherlab/ingest`: manifest readers and dataset adapters
- `src/decipherlab/vision`: synthetic glyph rendering and image-level helpers
- `src/decipherlab/transcription`: posterior representations and transcript utilities
- `src/decipherlab/glyphs`: image features, clustering, and inventory recovery baselines
- `src/decipherlab/structure`: structural diagnostics and null controls
- `src/decipherlab/hypotheses`: hypothesis-family scorers and evidence objects
- `src/decipherlab/scoring`: ranking and report generation
- `src/decipherlab/benchmarks`: synthetic data generation
- `src/decipherlab/evaluation`: metrics and benchmark runners
- `src/decipherlab/utils`: config loading, logging, seeds, artifact IO
- `configs/experiments`: validated YAML configs
- `docs`: research-facing design and protocol documents
- `tests`: unit and integration coverage

## Build, Test, And Validation Commands

- `uv sync`
- `uv run pytest`
- `uv run decipherlab demo synthetic-uncertainty`
- `uv run decipherlab run-pipeline --config configs/experiments/synthetic_mvp.yaml`
- `python3 scripts/run_core_claim_experiment.py --config configs/experiments/real_manifest_uncertainty.yaml`
- `PYTHONPATH=src python3 -m decipherlab.cli validate-manifest --manifest-path path/to/manifest.yaml --manifest-format glyph_crop`
- `python3 -m pytest`
- `python3 -m py_compile $(find src -name '*.py')`

## Coding Conventions

- Prefer minimal diffs and cohesive patches.
- Use typed Python where it improves interface clarity.
- Keep modules auditable. Avoid hidden side effects and implicit globals.
- Label heuristic or approximate methods explicitly in docstrings, reports, and comments.
- Separate reusable library code from experiment orchestration code.
- Prefer simple baseline features before introducing learned models.

## Scientific Conventions

- Keep scientific claims separate from engineering claims.
- Distinguish structure discovery, family classification, and semantic recovery.
- Never present fluency alone as evidence.
- Always compare against nulls, shuffled controls, or adversarial distractors when relevant.
- Treat unsupported or underdetermined outputs as valid outcomes.
- Record seeds, configs, and artifact paths for every experiment.

## Done Means

- Code paths are exercised by tests or an integration demo.
- Configs validate and produce deterministic outputs under a fixed seed.
- Reports describe caveats and heuristic components honestly.
- Public interfaces are documented in code and reflected in docs when behavior changes.
- New modules are either used immediately or clearly justified as extension points.

## Do-Not Rules

- Do not overclaim decipherment capability.
- Do not add black-box components without an auditable interface and a validation plan.
- Do not merge structural diagnostics and semantic interpretation into a single opaque score.
- Do not silently rewrite tracked artifacts in place.
- Do not accept hypothesis rankings without reporting evidence and uncertainty.

## Experiment Hygiene

- Keep raw inputs immutable.
- Write generated datasets under `data/synthetic/` and experiment outputs under `outputs/runs/`.
- Version configs by file and hash, not by memory or notebook state.
- Use fixed seeds unless the experiment explicitly studies stochastic variation.
- Record null baselines alongside primary scores.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toniIepure25) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
