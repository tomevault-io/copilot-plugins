## workflow-py

> Before committing any changes, always run:

# Agents

## Linting and type checking

Before committing any changes, always run:

```
poetry run ruff format .
poetry run ruff check .
poetry run mypy --show-error-codes .
```

Fix all errors before committing. Do not commit code with unused imports, formatting issues, or type errors.

---
> Source: [upstash/workflow-py](https://github.com/upstash/workflow-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
