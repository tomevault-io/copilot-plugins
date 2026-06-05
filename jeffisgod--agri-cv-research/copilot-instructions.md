## agri-cv-research

> This is an **automated agricultural computer vision research framework**. Users input a research direction and the system automatically performs literature search, hypothesis generation, experiment execution, result analysis, and paper writing.

# Agri-CV Research Framework — Build Notes

## Project Overview

This is an **automated agricultural computer vision research framework**. Users input a research direction and the system automatically performs literature search, hypothesis generation, experiment execution, result analysis, and paper writing.

## Build Order

Build phases are defined in `.cursor/rules/build-instructions.mdc` and `docs/agri-cv-auto-research-guide.md`.

**Current Phase: Phase 1 — Skeleton Setup**

## Key Conventions

- Python 3.10+, type hints on all functions, Google-style docstrings
- snake_case for variables, PascalCase for classes
- All configs in YAML (not JSON)
- All dataset loaders inherit `datasets/base.py::BaseDataset`
- All model wrappers inherit `models/base.py::BaseModel`
- All pipeline stages inherit `pipeline/base_stage.py::BaseStage`
- seed=42 for all experiments
- Citation verification is mandatory — no LLM fabrications

## Important Notes

- `CLAUDE.md` was missing at project root and has been recreated
- The `agri_cv_research/` package is the CLI entry point (`agri-research` command)
- Test files are at `tests/test_*.py`, run with `pytest tests/`
- See `docs/agri-cv-auto-research-guide.md` for the full specification

## Quick Commands

```bash
# Install in editable mode
pip install -e .

# Run CLI
agri-research --help
agri-research info

# Run tests
pytest tests/

# Current status
# Phase 1 complete: skeleton, pyproject.toml, requirements.txt, CLI, SKILL.md, README.md, CLAUDE.md
# Phase 2+ not yet started
```

---
> Source: [Jeffisgod/Agri-CV-Research](https://github.com/Jeffisgod/Agri-CV-Research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
