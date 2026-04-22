## rakfort-promptfoo2

> Guidelines for Python development


# Python Development Guidelines

Files: "\*_/_.py"

This rule provides guidance for Python development in the promptfoo project.

## Python Guidelines

- Use Python 3.9 or later
- Follow the [Google Python Style Guide](mdc:https://google.github.io/styleguide/pyguide.html)
- Use type hints to improve code readability and catch potential errors
- Use `ruff` for linting and formatting
  - Run `ruff check --fix` for general linting
  - Run `ruff check --select I --fix` for import sorting
  - Run `ruff format` for formatting
- Write unit tests for new Python functions using the built-in `unittest` module
- Keep the Python codebase simple and minimal, without unnecessary external dependencies

## Examples and Custom Components

- When adding Python examples, update relevant `requirements.txt` files
- When implementing custom providers, prompts, or asserts in Python, follow the promptfoo API patterns
- Place example Python code in the `examples/` directory with appropriate documentation

## Integration with TypeScript

- Understand the interaction between Python components and the TypeScript core
- Follow the patterns established in existing Python files for interfacing with the main codebase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/UncleWeeds) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
