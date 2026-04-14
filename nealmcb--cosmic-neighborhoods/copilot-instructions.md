## cosmic-neighborhoods

> - Python 3.10+ required

# Cosmic Neighborhood Project Rules

## Dependencies
- Python 3.10+ required
- Use Typer for CLI parsing
- Use Rich for output formatting
- No external API calls in v1

## Code Style
- Use black for formatting (line length 100)
- Use ruff for linting
- Use mypy --strict for type checking
- All functions must have type hints and docstrings
- Keep core.py pure computation, cli.py for interface

## Design Constraints
- No external API calls in v1
- Use simple algorithms/approximations
- Accuracy suitable for casual users
- Support both human-readable and JSON output
- Store no external state (v1)

## Testing
- All core functions must have unit tests
- Use pytest for testing
- Maintain test coverage

## Quality Checks
- Run make check before commits
- Ensure all linting passes
- Ensure all type checks pass
- Ensure all tests pass

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nealmcb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
