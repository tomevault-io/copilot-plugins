## autox-tools

> autox-tools: CLI tooling for Red Hat OpenShift AI's AutoRAG and AutoML features.

# AGENTS.md

## Project

autox-tools: CLI tooling for Red Hat OpenShift AI's AutoRAG and AutoML features.
Tools interact with vector databases, object storage, Kubeflow pipeline infrastructure, OGX AI gateways, and Kubernetes secrets.

## Quick orientation

- Entry points are registered in `pyproject.toml` under `[project.scripts]`.
- Each tool lives in its own subpackage under `autox_tools/`.
- Connection config is driven by environment variables via `python-dotenv`.
- Standard layout per tool: `cli.py` (argparse `main()`), `_client.py` (connection factory), `README.md`.

## Discovery commands

```bash
uv sync                          # Install dependencies
uv run <tool> --help             # Help for any registered entry point
uv run pytest                    # Run tests
uv run ruff check autox_tools/   # Lint
uv run mypy autox_tools/         # Type-check
```

## Adding a tool

1. Create `autox_tools/<name>/` with `__init__.py`, `cli.py`, and `_client.py`.
2. Register the entry point in `pyproject.toml` `[project.scripts]`.
3. Add dependencies to `pyproject.toml` `dependencies` list.
4. Add `autox_tools/<name>/README.md` with env-var docs and command reference.

## Upstream context

- [ai4rag](https://github.com/IBM/ai4rag) -- RAG optimization engine (OGX backend, GAMOptimizer)
- [pipelines-components](https://github.com/opendatahub-io/pipelines-components) -- Kubeflow pipeline components (OpenDataHub)
- [autox-ci](https://github.com/red-hat-data-services/autox-ci) -- E2E test suite with shared KFP/S3/diagnostics utilities

## Conventions

- Python 3.11+, managed with `uv` and `hatchling` build backend.
- `ruff` for linting (line-length 120; see `pyproject.toml` `[tool.ruff.lint]` for the full rule set).
- `mypy` for type checking (`check_untyped_defs` enabled).
- `argparse` for CLI interfaces -- not click or typer.
- Every CLI tool supports a `--json` global flag for machine-readable output.
- Credentials and endpoints via environment variables; never hardcoded.

---
> Source: [jakub-walaszczyk/autox-tools](https://github.com/jakub-walaszczyk/autox-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
