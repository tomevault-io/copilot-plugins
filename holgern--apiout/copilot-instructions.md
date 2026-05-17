## apiout

> This document provides guidelines for AI agents working on this repository.

# Agent Instructions for apiout

This document provides guidelines for AI agents working on this repository.

## Project Overview

Instructions:

- Never use git commands, you must not commit code

## Installation

### For Development

```bash
pre-commit run --all-files
```

## Documentation

### Building Documentation

The project documentation is built using Sphinx and hosted on ReadTheDocs.

To build the documentation locally:

```bash
cd docs && make clean && make html
```

The generated documentation will be available in the `docs/_build/html` directory.

### API Documentation

When working on the code, please follow these documentation guidelines:

- Use docstrings for all public classes, methods, and functions
- Follow the NumPy docstring format
- Include type hints in function signatures
- Document parameters, return values, and raised exceptions

## Development

### Running Tests

Run all tests using `pytest`.

To run tests with coverage: `pytest --cov=apiout --cov-report=term`

### Linting and Formatting

This project uses `ruff` for linting and formatting, and `prettier` for other file
types. These are enforced by pre-commit hooks.

Run linting and formatting:
`ruff check --fix --exit-non-zero-on-fix --config=.ruff.toml`

### cli

Do not use `python -m apiout` to run the cli but `apiout` directly!

### pip

Do not use `pip install` but `uv pip install`! apiout is install with `-e .` .

## Project Structure

The project is organized as follows:

```
apiout/                   # Main package
tests/                     # Test directory
docs/                      # Documentation
examples/                  # Example configurations and scripts
schemas/                   # Schema definitions
.github/                   # GitHub workflows
pyproject.toml             # Project metadata and build configuration
requirements.txt           # Package dependencies
requirements-test.txt      # Test dependencies
README.md                  # Project README
```

### Key Modules

- `apiout/__init__.py`: Package initialization and public API exports
- `apiout/cli.py`: Command-line interface implementation
- `apiout/fetcher.py`: API data fetching and HTTP client functionality
- `apiout/generator.py`: Data generation, processing, and transformation logic
- `apiout/serializer.py`: Data serialization and formatting functionality
- `schemas/apiout.json`: JSON Schema for TOML configuration validation

## Code Style

- **Formatting**: Adhere to the `ruff` and `prettier` configurations. Maximum line
  length is 88 characters.
- **Imports**: Follow the `isort` configuration in `.ruff.toml`. Imports are grouped
  into `future`, `standard-library`, `third-party`, `first-party`, and `local-folder`.
- **Naming**: Use `snake_case` for functions and variables, and `PascalCase` for
  classes.
- **Types**: Add type hints for all new functions and methods.
- **Error Handling**: Use standard `try...except` blocks for error handling.

## Contribution Guidelines

### TOML Schema

The project includes a JSON Schema for TOML validation located at `schemas/apiout.json`.

When adding new fields or properties to TOML configurations:

1. Update the schema in `schemas/apiout.json`
2. Add new properties, enums, or definitions as needed
3. Test with `taplo check examples/*.toml`
4. Update schema documentation in `schemas/README.md`

The schema provides:

- Validation for field definitions, formatters, and layouts
- Autocomplete in VS Code (with Even Better TOML extension)
- Type checking for TOML configuration files

### Common Issues

When working on the code, be aware of these common issues:

1. TOML configuration parsing: Ensure layout configurations follow the expected schema
   defined in `loader.py` Pydantic models and `schemas/apiout.json`
2. Field registry resolution: Fields are resolved first from the field registry, then
   from the context dictionary
3. Formatter parameters: Formatters accept optional parameters that should be validated
   and have sensible defaults
4. Type hints: Ensure proper typing for all functions, especially for context
   dictionaries and layout configurations
5. Schema updates: When adding new operations, transforms, or validation types, update
   the JSON schema

---
> Source: [holgern/apiout](https://github.com/holgern/apiout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
