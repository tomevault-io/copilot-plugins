## dbt-semantic-interfaces

> This project uses **hatch** for dependency management and testing:

# `dbt-semantic-interfaces` Project Rules

This project uses **hatch** for dependency management and testing:

- Dependencies: Use `hatch` commands, not `pip install` directly.
- Linting: Run `make lint` to detect and fix lint errors.
- Testing:
  - Use `hatch run dev-env:pytest <path>` to run tests in a specific file.
  - Use `make test` to run all tests.

* Don't add obvious comments.

## Python Code Standards

- **Always add type annotations** to all Python functions (parameters and return types)
- Use `from __future__ import annotations` at the top of Python files

---
> Source: [dbt-labs/dbt-semantic-interfaces](https://github.com/dbt-labs/dbt-semantic-interfaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
