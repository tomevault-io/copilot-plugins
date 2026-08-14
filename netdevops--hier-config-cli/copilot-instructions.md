## hier-config-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

hier-config-cli is a Click-based CLI tool for network configuration analysis built on the `hier-config` library. It compares running and intended device configs to generate remediation, rollback, and future-state configurations. Supports 10 network platforms (Cisco IOS/NXOS/XR, Arista EOS, Juniper JunOS, VyOS, FortiOS, HP Comware5/ProCurve, Generic).

## Build & Development Commands

```bash
# Install dependencies
poetry install

# Run all tests with coverage
pytest

# Run a single test
pytest tests/test_cli.py::test_version

# Code formatting
black src/ tests/

# Linting
ruff check src/ tests/

# Type checking
mypy src/

# All quality checks
black src/ tests/ && ruff check src/ tests/ && mypy src/ && pytest
```

## Architecture

The entire CLI lives in a single module: `src/hier_config_cli/__main__.py`. The entry point is the `cli()` Click group, registered as `hier-config-cli` in pyproject.toml.

**Key flow:** All three config commands (`remediation`, `rollback`, `future`) share the same pattern:
1. `common_options` decorator applies shared Click options (platform, running-config, generated-config, format, output)
2. `process_configs()` validates the platform, reads both config files via `hier_config.utils.read_text_from_file`, parses them with `get_hconfig()`, then runs the requested operation via `WorkflowRemediation` (for remediation/rollback) or `HConfig.future()` (for future)
3. `format_output()` converts the result HConfig to text/JSON/YAML
4. `get_output_text()` handles platform-specific formatting — JunOS uses `line.text`, all others use `line.cisco_style_text()`

**PLATFORM_MAP** dict maps CLI string names to `hier_config.Platform` enum values. To add a new platform, add it here, handle any special output formatting in `get_output_text()`, and add tests.

## Code Standards

- Python 3.10+ (uses `X | Y` union syntax)
- Black formatter, 100-char line length
- Ruff linter rules: E, W, F, I, B, C4, UP
- mypy strict mode (all functions must have type hints)
- Google-style docstrings
- Conventional commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`

## Testing

Tests use Click's `CliRunner` for CLI invocation testing. Fixtures provide mock Cisco IOS and Juniper JunOS configs via temporary files. The test suite covers all commands, output formats, all platforms, error handling, and logging levels.

## CI/CD

- `test-app.yaml`: Runs black, ruff, mypy, and pytest on Python 3.10–3.13 for every push/PR to main
- `deploy.yaml`: Publishes to PyPI via Poetry on GitHub release creation
- Version is maintained in both `pyproject.toml` and `__main__.py:__version__`

---
> Source: [netdevops/hier-config-cli](https://github.com/netdevops/hier-config-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
