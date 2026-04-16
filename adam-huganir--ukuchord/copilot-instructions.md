## ukuchord

> This is a Python music theory library focused on ukulele chord generation.


# Windsurf Rules for ukuchord

## Project Overview
This is a Python music theory library focused on ukulele chord generation.


## Comments and Docstrings
- Comment only when code is not self-explanatory. 
- Use simple language rather than long winded explanations.
- Assume that anyone reading the comments is at least a competent programmer so comments should be concise.

## Code
- Follow the standard PEP 8 Python style guidelines
- Use google style docstrings for all public methods
- Use type hints for all function parameters and return values
- Use descriptive variable and function names
- Keep functions focused and single-purpose, and avoid coupling unrelated tasks to help with refactoring.
- Code should be readable, but prioritize performance and concise code over readability.

## Testing
- Any internal function should have tests as pytest unit tests under ./tests.
- Any public functionality should have tests as pytest BDD tests under ./features.
- Follow existing BDD patterns in features/ directory
- Test edge cases for musical intervals and note parsing

## Environment
- Use `uv` to manage the environment.
- Use `uv` to run the application.
- Use `uv` to run the tests.
- `uv` is found in /home/adam/.local/bin so make sure that is added to the PATH

## Dependencies
- Use `uv add` to add new dependencies. Do not edit the pyproject file directly to add dependencies.
- Use uv for dependency management (uv.lock exists)
- Follow pyproject.toml configuration
- Respect existing ruff linting rules

## File Organization
- Core music theory in ukuchord/ directory
- Instrument-specific code in ukuchord/instruments/
- Unit tests in tests/ directory
- BDD tests in features/ directory
- Keep related functionality grouped together

## Music Theory Conventions
- Maintain consistency of terminology across the codebase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adam-huganir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
