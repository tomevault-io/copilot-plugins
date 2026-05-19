## oasis-tests-pipx-only

> Enforce pipx-only test execution workflow


# OASIS tests: pipx only

- Always run tests through `pipx` from the repository root.
- **NEVER run tests outside `pipx`.**
- **NEVER use `uv` for test execution in this repository.**
- **NEVER use system Python for tests** (`python -m unittest ...`, `python -m pytest ...`).
- **NEVER use global/local non-pipx pytest/unittest flows** (including direct `pytest` commands outside `pipx run`).
- Required command pattern:

```bash
PYTHONPATH="$(pwd)" pipx run --spec . python -m unittest tests.<module_or_class>
```

- For broader verification, run multiple test modules in one pipx invocation.
- Example broader verification command:

```bash
PYTHONPATH="$(pwd)" pipx run --spec . python -m unittest tests.test_oasis_cli tests.test_embedding_pure tests.test_report_schema
```

---
> Source: [psyray/oasis](https://github.com/psyray/oasis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
