## ml-cursor-intro

> - `ML_Intro/` hosts the teaching notebooks; templates mirror each lesson (`template_*`) so keep source and template notebooks in sync when updating exercises.

# Repository Guidelines

## Project Structure & Module Organization
- `ML_Intro/` hosts the teaching notebooks; templates mirror each lesson (`template_*`) so keep source and template notebooks in sync when updating exercises.
- `python-test/` contains the LangChain arithmetic tools (`langchain_math_tools.py`) and a runnable demo (`main.py`) that can double as a smoke test for new utilities.
- Top-level PDFs document the course context; update them only when lecture materials change and note the revision in the commit message.

## Environment & Setup
- Target Python 3.10+ and isolate work in a virtual environment (`python -m venv .venv && source .venv/bin/activate`).
- Install the minimal toolchain before running notebooks or scripts: `pip install jupyter langchain numpy pandas scikit-learn torch`.
- Use `jupyter lab` (or `jupyter notebook`) from the repo root so relative paths inside notebooks resolve correctly.

## Build, Test, and Development Commands
- `python python-test/main.py` — Runs the LangChain demo and verifies the arithmetic tools load and execute.
- `jupyter lab` — Launches an interactive workspace for modifying notebooks in `ML_Intro/`.
- `python -m compileall python-test` — Quick syntax check for added Python modules before committing.

## Coding Style & Naming Conventions
- Follow PEP 8: four-space indentation, `snake_case` for functions and variables, `CapWords` for classes.
- Keep functions small and type-annotated, mirroring the existing helpers in `python-test/langchain_math_tools.py`.
- For notebooks, begin with a Markdown overview cell, tag important results, and reset/execute all cells before committing.

## Testing Guidelines
- No automated suite exists yet; when adding logic, pair each new tool with a reproducible example in `python-test/main.py` or a dedicated notebook cell.
- Prefer pytest for future test modules placed under `python-test/tests/`; run with `pytest -q` and document any external service mocks.
- Manually rerun affected notebooks end-to-end to confirm outputs regenerate without hidden state.

## Commit & Pull Request Guidelines
- Keep commit subjects concise and action-oriented (e.g., `add knn template walkthrough`, `refactor langchain demo`), matching the existing history.
- Group notebook and script changes separately when possible, and include bullet summaries plus before/after visuals or metrics in pull request descriptions.
- Reference related issues or lesson plans, and attach notebook execution evidence (screenshots or cell output hashes) when behavior changes.

---
> Source: [ironmanciti/ML_Cursor_Intro](https://github.com/ironmanciti/ML_Cursor_Intro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
