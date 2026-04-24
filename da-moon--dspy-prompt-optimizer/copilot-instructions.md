## dspy-prompt-optimizer

> Core project context and architecture for dspy-prompt-optimizer


# DSPy Prompt Optimizer Workspace

## Project Overview
This is a Python library project for optimizing prompts in DSPy (Declarative Self-improving Python). The project uses Poetry for dependency management and follows strict typing and quality standards.

## Core Architecture
- **Package Manager**: Poetry
- **Python Version**: 3.11+
- **Main Package**: `prompt_optimizer/`
- **Testing**: pytest with comprehensive coverage
- **Quality Tools**: pyright, mypy, pylint, sourcery

## Key Directories
- `prompt_optimizer/`: Main source code package
- `tests/`: Test suite
- `prompts/`: Prompt templates and examples
- `typings/`: Custom type definitions
- `.cursor/rules/`: Cursor IDE rules (this directory)

## Development Environment
- Virtual environment managed by Poetry (`.venv/`)
- Configuration files: `pyproject.toml`, `poetry.toml`
- Type checking: `pyrightconfig.json`, `pyrightconfig.strict.json`

## Core Constraints
- All code must pass strict type checking (pyright + mypy)
- 100% type annotation coverage required
- No `# type: ignore` comments allowed
- Conventional commits required
- Quality over speed - all checks must pass before proceeding

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/da-moon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
