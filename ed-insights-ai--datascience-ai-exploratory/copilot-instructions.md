## datascience-ai-exploratory

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build/Test Commands
- Environment setup: Check project-specific README.md or pyproject.toml for setup instructions
- Install dependencies: `uv sync` (for UV-based projects)
- Run tests: Refer to project-specific test instructions, typically `pytest tests/`
- Execute notebooks: `jupyter nbconvert --execute --to notebook --inplace path/to/notebook.ipynb`
- Run CLI scripts: Check each project's documentation for specific run commands

## Code Style Guidelines
- Python: Follow PEP 8 conventions
- Type hints: Use Python type annotations for all function parameters and return values
- Imports: Organize imports by standard library, third-party, then local modules
- Error handling: Use try/except with specific exception types and descriptive messages
- Paths: Make file paths relative to package location for portability
- Documentation: Include docstrings for all modules, classes, and functions (Google style)
- Defensive coding: Implement null checks and validation for all inputs
- Testing: Include unit tests with proper assertions for all components
- Agent code: Follow LangChain best practices for agent implementations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ed-insights-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
