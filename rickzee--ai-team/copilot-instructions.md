## python-style

> Python code style and conventions for ai-team


# Python Style

- **Formatter**: black, line length 100
- **Linter**: ruff (E, F, I, N, W, UP, B, C4, SIM)
- **Type checker**: mypy strict for `src/`
- **Line length**: 100 characters max
- **Naming**: snake_case (functions/variables), PascalCase (classes), UPPER_SNAKE (constants)
- **Imports**: isort profile=black; groups: stdlib → third-party → local
- **Docstrings**: Google style on all public classes, methods, and functions
- **Type hints**: required on all function signatures; use `from __future__ import annotations` in every file

## File Header

Every Python file must start with a module docstring:

```python
"""
Module name and one-line description.

Detailed description of what this module does and how it fits into the system.
"""
```

---
> Source: [RickZee/ai-team](https://github.com/RickZee/ai-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
