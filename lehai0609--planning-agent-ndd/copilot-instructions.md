## planning-agent-ndd

> - **Install dependencies**: `uv sync` or `uv sync --extra dev --extra nlp --extra ml`

# AGENTS.md

## Build, Lint, and Test Commands

- **Install dependencies**: `uv sync` or `uv sync --extra dev --extra nlp --extra ml`
- **Run linter**: `ruff check .` or `black --check .`
- **Format code**: `ruff format .` or `black .`
- **Run tests**: `pytest` or `pytest tests/` (if tests directory exists)
- **Run single test**: `pytest tests/test_<module>.py::test_<function_name>`
- **Run notebook**: Open `notebooks/draft.ipynb` in Jupyter or VS Code

## Architecture & Codebase Structure

This is a **notebook-driven development (NDD)** audit planning RPA system. Core workflow is in `notebooks/draft.ipynb` (synced with `draft.py` via jupytext).

- **`notebooks/`**: Main development workspace; contains Jupyter notebooks and `.py` pairs
- **`data/`**: Raw input files (TB, GL Excel files, audited FS PDFs)
- **`configs/`**: YAML configuration files (e.g., `tomoe_2025.yaml`) defining client/year parameters
- **`artifacts/`**: Generated outputs (parquet files, run logs, manifests)
- **Workflow**: Config load → Data ingestion → Validation (pandera) → FS mapping → Analytics → Visualization

## Code Style & Conventions

- **Python version**: 3.12+
- **Line length**: 100 characters (enforced by ruff/black)
- **Imports**: Standard library → Third-party → Local, grouped and sorted
- **Typing**: Use type hints (`from typing import Any, Dict, List, Optional`)
- **Data validation**: Use `pandera` schemas for DataFrame validation
- **Paths**: Use `pathlib.Path` for all file operations
- **Logging**: Append structured JSON logs to `artifacts/run_log.json`
- **Naming**: snake_case for functions/variables, PascalCase for classes, UPPER_CASE for constants
- **Error handling**: Explicit validation with pandera; log failures to run_log
- **Data formats**: Use parquet for intermediate datasets; Excel/CSV for raw inputs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lehai0609)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lehai0609)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
