## testing

> RaQuet Help


- This is a Python project
- Expect that a virtualenv .venv directory already exists and use it when it does
- The raquet module and requirements are described in pyproject.toml
- When we run tests in this project, we use pytest with this command: `python -m pytest -m "not integration"`
- The project linter is ruff, new commits should always pass lint:
    - `python -m ruff check raquet/*.py tests/*.py`
    - `python -m ruff format raquet/*.py tests/*.py`

---
> Source: [CartoDB/raquet](https://github.com/CartoDB/raquet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
