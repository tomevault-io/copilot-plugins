## mesoor-space-api-package

> This file provides guidance for agentic coding agents operating in this repository.

# AGENTS.md

This file provides guidance for agentic coding agents operating in this repository.

Repository root: `/Users/chenjiabin/mesoor-space-api-package`

---

## 1. Project Overview

- Language: Python (>= 3.9)
- Packaging: setuptools (`setup.py` based)
- Type: Async API client library
- Core dependencies: `aiohttp`, `pydantic`, `tenacity`
- Distribution artifacts: `sdist` and `wheel`

Primary package directory:

```
space_api/
```

Key modules:

- `space_api/mesoor_space.py`
- `space_api/exceptions.py`
- `space_api/model/`

There is no CI, no formal test suite, and no lint/format configuration.

---

## 2. Environment Setup

Create a virtual environment (recommended):

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Install in editable mode (recommended for development):

```bash
pip install -e .
```

---

## 3. Build Commands

This project uses traditional setuptools via `setup.py`.

Build source and wheel distributions:

```bash
python3 setup.py sdist bdist_wheel
```

Artifacts will be generated in:

- `dist/`
- `build/`

Clean build artifacts manually if needed:

```bash
rm -rf build dist *.egg-info
```

---

## 4. Test Commands

There is **no automated test framework** configured (no pytest, unittest suite, tox, or nox).

The only example test-like script:

```
space_api/example/test.py
```

Run the example script:

```bash
python space_api/example/test.py
```

There is no concept of:

- Running all tests
- Running a single test file
- Running a single test case

If adding tests, prefer `pytest` and document commands explicitly.

---

## 5. Linting and Formatting

There are no configured tools for:

- Formatting (no black, autopep8)
- Linting (no flake8, ruff, pylint)
- Type checking (no mypy config)
- Import sorting (no isort)

Agents should follow standard modern Python style (PEP 8) unless the repository evolves to enforce specific tooling.

Do not introduce formatting-only changes unless requested.

---

## 6. Code Style Guidelines

### Imports

- Use absolute imports within the package (e.g., `from space_api.model import ...`).
- Group imports in this order:
  1. Standard library
  2. Third-party
  3. Local package imports
- Avoid wildcard imports.

### Formatting

- Follow PEP 8.
- 4 spaces per indentation level.
- Keep line length <= 100 characters when practical.
- Use explicit async/await patterns consistently.

### Types

- The project uses `pydantic` models in `space_api/model/`.
- Prefer explicit type hints for:
  - Function parameters
  - Return types
  - Public APIs
- Use built-in generics (`list[str]`, `dict[str, Any]`) for Python >= 3.9.

### Naming Conventions

- Modules: `snake_case`
- Functions: `snake_case`
- Variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`

Match the existing naming patterns in `space_api/`.

### Error Handling

- Custom exceptions live in `space_api/exceptions.py`.
- Prefer raising domain-specific exceptions instead of generic `Exception`.
- Preserve async stack traces; do not swallow exceptions silently.
- Use `tenacity` for retry logic where appropriate.

### Async Conventions

- All network operations should be `async`.
- Do not block the event loop.
- Avoid synchronous I/O inside async functions.
- Use `asyncio.run()` only in top-level scripts, never inside library code.

---

## 7. Packaging Guidelines

- Keep `setup.py` as the single source of packaging configuration.
- Update `install_requires` if adding runtime dependencies.
- Keep version updates consistent in `setup.py`.

Do not introduce `pyproject.toml` unless explicitly requested.

---

## 8. Repository Hygiene

- Do not modify generated directories:
  - `build/`
  - `dist/`
  - `*.egg-info/`
- Avoid committing virtual environments.
- Avoid adding unrelated formatting-only diffs.

---

## 9. AI Tooling Rules

There are no repository-specific AI rules:

- No `.cursor/rules/`
- No `.cursorrules`
- No `.github/copilot-instructions.md`

Agents should rely on this `AGENTS.md` as the primary operational guide.

---

## 10. Recommended Improvements (Optional)

If modernization is requested, suggested additions:

1. Add `pytest` with basic async test coverage.
2. Add `pyproject.toml`.
3. Add `ruff` or `black`.
4. Add CI (GitHub Actions).

Do not implement these unless explicitly instructed.

---

This repository is a minimal async Python client library. Prioritize correctness, clarity, and preserving existing behavior when making changes.

---
> Source: [chyaoyuan/mesoor-space-api-package](https://github.com/chyaoyuan/mesoor-space-api-package) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
