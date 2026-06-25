## mdfactory

> This file applies to the entire repository unless overridden by a more specific `AGENTS.md`.

# Agent Notes for mdfactory

## Scope
This file applies to the entire repository unless overridden by a more specific `AGENTS.md`.

## Project Summary
`mdfactory` is a tool for high-throughput setup, execution, and analysis of MD simulations, currently with a focus on lipid chemistry (membranes, LNPs, etc.). Core source lives under `mdfactory/`.

## Code Style
- Docstrings: NumPy style.
- Formatting/linting: enforced via pre-commit hooks.

## Testing
- Short tests: `pytest -k "not build" mdfactory`

## Repository Layout
- Core library: `mdfactory/`
- Tests: `mdfactory/tests/`
- Configuration: `pyproject.toml`

## Versioning
The version is a static string in `mdfactory/__init__.py` (`__version__ = "x.y.z"`), read at build time by hatchling. To bump it:
```bash
pixi run -e dev bump-my-version bump patch  # 0.1.0 → 0.1.1
pixi run -e dev bump-my-version bump minor  # 0.1.0 → 0.2.0
pixi run -e dev bump-my-version bump major  # 0.1.0 → 1.0.0
```
This updates `__init__.py`, creates a commit, and tags it `v<new_version>`.

## Analyses and Artifacts
- Analyses and artifacts run through `mdfactory.analysis.simulation.Simulation`, which
  dispatches registered functions via `ANALYSIS_REGISTRY` and `ARTIFACT_REGISTRY`.
- Analysis outputs are stored as parquet in `.analysis/<analysis_name>.parquet` and
  registered in `.analysis/metadata.json` alongside column metadata.
- Artifact outputs are stored under `.analysis/artifacts/<artifact_name>/` and are also
  recorded in the registry with file checksums and paths.
- Batch utilities live in `mdfactory/analysis/store.py` (status, batch run, removal),
  while submitit-backed SLURM submission is in `mdfactory/analysis/submit.py`.

---
> Source: [emdgroup/mdfactory](https://github.com/emdgroup/mdfactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
