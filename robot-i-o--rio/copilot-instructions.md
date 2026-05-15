## rio

> **Always use `uv run`, not python**.

# Development Workflow

**Always use `uv run`, not python**.

We've bundled common commands into a Makefile for convenience.

```sh
make format     # Format and lint
make test
```

Always run `make check` before committing. This runs formatting, linting,
and type checking. Do not commit code that fails type checking.

Before creating a PR, ensure all checks pass with `make test`.

## Code style

- **When creating a new node, start from the node template.**
  - copy over `_template/template.py` to `submodule/my_module.py`
- **snake_case.py for file names and PascalCase for class names.**
  - `submodule/my_module.py` contains `MyModule` class.
  - avoid fully capitalized acronyms, `UsbCamera` (not `USBCamera`).
- **Prefix-first naming for discoverability (autocomplete).**
  - **Methods**: `add_shape_sphere()` (not `add_sphere_shape()`).
- **Method names are `snake_case`.**
- **Prefer nested classes when self-contained.**
  - If a helper type or an enum is only meaningful inside one parent class and doesn't need a public identity, define it as a nested class instead of creating a new top-level class/module.
- **Follow PEP 8 for Python code.**
- **Use modern Python type-hint syntax.**
  - Prefer PEP 604 unions: `x | y`, `x | None`. Do not use `typing.Union` or `typing.Optional`.
- **Use Google-style docstrings.**
  - Write clear, concise docstrings that explain what the function does, its parameters, and its return value.
  - Keep argument/return types in function annotations, not inline in docstrings.
  - In `Args:` entries, use `name: description` (not `name (Type): description`).
- **Avoid new required dependencies.** Strongly prefer not adding optional ones, use existing dependencies like numpy or stdlib.

## Commit and Pull Request Guidelines

Follow conventional commit message practices.
- Use clear, descriptive commit messages that explain what changed and why.
- Keep commits focused and atomic—one logical change per commit.
- Reference related issues in commit messages when applicable.

For detailed guidance on writing good commit messages and structuring pull requests, see [Apache Airflow's Pull Request Guidelines](https://github.com/apache/airflow/blob/main/AGENTS.md#pull-request-guidelines).

---
> Source: [robot-i-o/rio](https://github.com/robot-i-o/rio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
