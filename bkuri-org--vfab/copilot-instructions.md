## vfab

> - **Install**: `uv pip install -e ".[dev,vpype]"` (add `,axidraw` for hardware)

# vfab Agent Guidelines

## Build/Test Commands
- **Install**: `uv pip install -e ".[dev,vpype]"` (add `,axidraw` for hardware)
- **Lint**: `uvx ruff check .`
- **Format**: `uvx black .`
- **Test single**: `uv run pytest tests/test_<module>.py -q`
- **Test all**: `uv run pytest -q`
- **DB migrate**: `uv run alembic upgrade head`
- **Pre-commit**: `uvx pre-commit install && uvx pre-commit run -a`

## Performance & QA Testing (v0.8.0+)
- **Memory Profiling**: `uv run python tests/test_memory_simple.py`
- **Database Performance**: `uv run python tests/test_database_performance.py`
- **Cross-Platform Test**: `uv run python tests/test_cross_platform.py`
- **Load Testing**: `uv run python tests/test_load.py` (add `--quick` for fast test)
- **Final QA**: `uv run python tests/test_final_qa.py`
- **Release Validation**: `uv run python scripts/validate_release.py`
- **Release Automation**: `uv run python scripts/release.py <version>`

## Code Style
- **Python**: 3.11+, use `from __future__ import annotations`
- **Imports**: Group stdlib, third-party, local imports; use `typer`, `pydantic`, `pathlib`
- **Formatting**: Black with default settings
- **Types**: Use Pydantic models for config, type hints everywhere
- **Naming**: snake_case for functions/variables, PascalCase for classes
- **Error handling**: Use `typer.BadParameter` for CLI validation, `SystemExit` for fatal errors
- **CLI**: Use Typer for commands, `no_args_is_help=True`
- **Config**: YAML-based with Pydantic validation in `config.py`
- **Tests**: pytest with descriptive docstrings, Mock for external deps

---
> Source: [bkuri-org/vfab](https://github.com/bkuri-org/vfab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
