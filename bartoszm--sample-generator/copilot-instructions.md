## sample-generator

> - Use tools: `black`, `ruff`, `isort`.


# Python 3.12 Best Practices for Juniors & Copilots

## 1. Code Style
- Follow **PEP 8**.
- Use tools: `black`, `ruff`, `isort`.
- Use **snake_case** for variables/functions.
- Limit lines to **≤ 79 characters**.

## 2. Type Hints
- Use **type annotations**:
  ```python
  def greet(name: str) -> str:
      return f"Hello, {name}"
  ```
- Add `from __future__ import annotations` for forward references.

## 3. Immutability & Constants
- Prefer **immutable types** (`tuple`, `frozenset`).
- Define constants in **ALL_CAPS**.

## 4. Structure
- Organize into **functions**, **classes**, **modules**.
- Follow **Single Responsibility Principle** for modules.

## 5. Exceptions
- Catch **specific exceptions**:
  ```python
  try:
      result = 1 / x
  except ZeroDivisionError:
      handle()
  ```
- Avoid bare `except:` or broad `except Exception:`.

## 6. Modern Python Features
- Use **match-case** (structural pattern matching):
  ```python
  match command:
      case "start":
          start()
      case "stop":
          stop()
  ```
- Use **walrus operator** (`:=`) for inline assignments.

## 7. Data Handling
- Use `@dataclass`, `NamedTuple`, or `attrs` for structured data.
- Prefer **generators** over lists for large/streamed data.

## 8. Imports
- Keep imports **explicit and grouped**:
  ```python
  # Standard library
  import os

  # Third-party
  import requests

  # Local
  from .utils import helper
  ```
- Avoid `from module import *`.

## 9. Testing
- Use `pytest` or `unittest`.
- Cover edge cases and use **asserts with messages**.
- Aim for **fast, deterministic, and isolated** tests.

## 10. Linting & CI
- Use tools: `ruff`, `mypy`, `black`, `pytest`.
- Run them in **pre-commit hooks** and **CI pipelines**.

```

Let me know if you want it styled for a GitHub README or as a team wiki doc.

---
> Source: [bartoszm/sample_generator](https://github.com/bartoszm/sample_generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
