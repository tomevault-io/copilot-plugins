## python-style

> Follow project style guidelines for Python code


# Python Coding Rules
- Use python 3.12 conventions: use `type | None` instead of `Optional` etc
- Use type hints everywhere (PEP 484).
- Use snake_case for functions and variables, PascalCase for classes.
- Use `pydantic.BaseModel` for structured data, not dataclasses
- Avoid unused imports.
- Use `async`/`await` where appropriate.
- Always keep functions small and composable.
- Avoid using hasattr at all costs, it's not safe

---
> Source: [growgraph/ontocast](https://github.com/growgraph/ontocast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
