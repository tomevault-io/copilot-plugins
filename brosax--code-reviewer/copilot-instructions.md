## code-reviewer

> Core pipeline code lives in `app/`. Use `app/adapters/` for tool-specific report importers, `app/ai/` for prompt and provider logic, `app/merge/` for deduplication/consolidation, `app/parsers/` for normalization, `app/reports/` for output rendering, and `app/utils/` for shared helpers. Tests live in `tests/`, with end-to-end coverage centered in `tests/test_pipeline.py`. Example configs, sample reports, and demo output live under `examples/`. Workflow notes are in `docs/workflow.md`, and plugin packaging lives in `plugins/code-review-assistant/`.

# Repository Guidelines

## Project Structure & Module Organization
Core pipeline code lives in `app/`. Use `app/adapters/` for tool-specific report importers, `app/ai/` for prompt and provider logic, `app/merge/` for deduplication/consolidation, `app/parsers/` for normalization, `app/reports/` for output rendering, and `app/utils/` for shared helpers. Tests live in `tests/`, with end-to-end coverage centered in `tests/test_pipeline.py`. Example configs, sample reports, and demo output live under `examples/`. Workflow notes are in `docs/workflow.md`, and plugin packaging lives in `plugins/code-review-assistant/`.

## Build, Test, and Development Commands
Install locally with `python -m pip install -e .` to expose the `review` CLI. Run the full pipeline with `python -m app.main all --config examples/review_config.yaml` or `review all --config examples/review_config.yaml`. Execute tests with `python -m unittest discover -s tests -v`. For a narrower loop, run a single module such as `python -m unittest tests.test_pipeline`.

## Coding Style & Naming Conventions
Target Python 3.11+ and follow the existing style: 4-space indentation, explicit type hints, small functions, and `dataclass(slots=True)` for shared models where appropriate. Use `snake_case` for modules, functions, variables, and config keys; use `PascalCase` for classes. Keep CLI command names hyphenated only at the argument layer, as in `first-pass`, then map them internally to Python identifiers. No formatter or linter is configured in `pyproject.toml`, so keep changes consistent with surrounding code and existing import ordering.

## Testing Guidelines
Tests use the standard library `unittest` framework and create temporary workspaces under `tests/`. Add coverage for new pipeline stages, config validation, and report transformations. Prefer test names that describe behavior, such as `test_collect_normalizes_reports`. Use the sample assets in `examples/` instead of introducing large new fixtures unless required.

## Commit & Pull Request Guidelines
Git history is not available in this workspace, so follow a simple convention: use short, imperative commit subjects such as `Add coverity severity mapping`. Keep commits focused on one change. Pull requests should summarize the affected pipeline stage, list validation performed, reference any related issue, and include sample output paths or screenshots when report formatting changes.

## Configuration & Assets
Keep sensitive data out of committed configs. Reference documents in `sesip source/` and generated artifacts in `examples/review-output/` are large, so avoid duplicating them unless the workflow genuinely changes.

---
> Source: [Brosax/Code_reviewer](https://github.com/Brosax/Code_reviewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
