## codesnap

> <!-- Source: .ruler/AGENTS.md -->



<!-- Source: .ruler/AGENTS.md -->

# CodeSnap Agent Guidelines

## Build/Lint/Test Commands
- **Install**: `uv sync`
- **Run tests**: `uv run pytest -v`
- **Run single test**: `uv run pytest -v -k "test_name"`
- **Type checking**: `uv run mypy src/`
- **Linting**: `uv run ruff check src/`
- **Formatting**: `uv run ruff format src/`

## Code Style Guidelines
- **Imports**: Group stdlib, third-party, local imports with blank lines
- **Types**: Use Pydantic models for data structures, type hints everywhere
- **Naming**: snake_case for variables/functions, PascalCase for classes
- **Error handling**: Use custom exceptions, rich error messages
- **CLI**: Click framework with rich output formatting
- **Formatting**: Follow ruff defaults, max line length 88
- **Documentation**: Google-style docstrings, type hints in function signatures

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zwldarren)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zwldarren)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
