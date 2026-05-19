## tests

> You are working on **tests** for the Hailo Apps repository.


# Tests — Contextual Instructions

You are working on **tests** for the Hailo Apps repository.

## Required Context Files

Read these files before making changes (paths relative to `.hailo/`):

- `instructions/testing-patterns.md` — pytest framework, markers, fixtures, patterns
- `skills/hl-validate.md` — 5 validation levels, test templates
- `instructions/coding-standards.md` — Import rules, logging conventions
- `memory/common_pitfalls.md` — Known anti-patterns to avoid

## Key Patterns for Tests

- Use `pytest` with markers: `@pytest.mark.pipeline`, `@pytest.mark.standalone`, `@pytest.mark.gen_ai`
- Test config in `hailo_apps/config/test_definition_config.yaml`
- Control which tests run via `tests/test_control.yaml`
- Use fixtures from `tests/conftest.py`
- Always use absolute imports in tests

## Running Tests

```bash
pytest tests/test_runner.py -v              # Pipeline app tests
pytest tests/test_standalone_runner.py -v   # Standalone tests
pytest tests/test_gen_ai.py -v              # GenAI tests (Hailo-10H only)
pytest tests/test_sanity_check.py -v        # Sanity checks
pytest tests/ -v                            # All tests
```

## Key Test Files

- `tests/conftest.py` — Shared fixtures and configuration
- `tests/test_runner.py` — Pipeline app test runner
- `tests/test_standalone_runner.py` — Standalone app tests
- `tests/test_gen_ai.py` — Gen AI app tests
- `tests/test_utils.py` — Test utility functions

---
> Source: [hailo-ai/hailo-apps](https://github.com/hailo-ai/hailo-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
