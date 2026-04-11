## coco

> > **Directive:** Deliver production-ready, highly-performant, and rigorously tested Python code with absolute confidence. As a veteran engineering team, we prioritize maintainability, readability, and robustness.

# Python Coding Guidelines & Team Standards

> **Directive:** Deliver production-ready, highly-performant, and rigorously tested Python code with absolute confidence. As a veteran engineering team, we prioritize maintainability, readability, and robustness.

## 1. Code Quality & Style

* **PEP 8 Compliance:** Adhere strictly to PEP 8. We use automated formatters and linters (like Ruff or Black/Flake8) to enforce this. Do not argue with the formatter.
* **Type Hinting:** Enforce strict type hints (`typing` module) for all function/method signatures and class attributes. Code must pass `mypy --strict` without errors.
* **Docstrings:** Use Google-style or NumPy-style docstrings for all modules, classes, and public functions. Briefly explain the *why* and the *what*, document all arguments, and specify return types and raised exceptions.
* **Naming Conventions:**
  * Variables, functions, methods: `snake_case`
  * Classes, Exceptions: `PascalCase`
  * Constants: `UPPER_SNAKE_CASE`
  * Protected/Private members: Prefix with a single underscore `_` (e.g., `_internal_method`).

## 2. Architecture & Design

* **Clarity Over Cleverness:** Write clean, self-documenting code. If a one-liner takes more than 10 seconds to understand, rewrite it as multiple lines.
* **Defensive Programming:** Fail fast. Validate inputs at the boundaries of your system. Handle exceptions gracefully, and *never* use bare `except:` or swallow exceptions silently (`pass`).
* **SOLID Principles:** Keep functions small and focused on a single responsibility. Favor composition over inheritance.
* **Dependency Management:** Explicitly declare all dependencies. Never assume a package is globally available. Use environments (`uv`, `poetry`, or `venv`).

## 3. High Performance

* **Algorithmic Efficiency:** Optimize for time and space complexity. Choose the right data structures (e.g., `set` for lookups, `collections.deque` for queues).
* **Resource Management:** Utilize context managers (`with` statements) for file I/O, database connections, and locks to ensure resources are always released properly.
* **Generators & Iterators:** Use generators (`yield`) for processing large datasets to minimize memory footprint.

## 4. Testing & Validation

* **Comprehensive Coverage:** Write unit, integration, and property-based tests for all new logic. We use `pytest`.
* **Edge Cases:** Test boundaries, nulls/Nones, limits, and unexpected inputs relentlessly.
* **Zero Flakiness:** Ensure tests are completely deterministic and isolated. Do not rely on external network calls or shared mutable state in unit tests; use mocking/patching appropriately (`unittest.mock` or `pytest-mock`).
* **Test Placement:** Keep tests in the `tests/` directory, mirroring the structure of the `src/` directory.

## 5. Version Control & Reviews

* **Atomic Commits:** Make small, logical, and independent commits.
* **Descriptive Messages:** Write clear commit messages. The first line should be a concise summary (max 50 chars), followed by a blank line, and then detailed context if necessary.
* **Code Review:** All code must be reviewed before merging. Reviewers should check for logical correctness, adherence to these guidelines, and adequate test coverage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pagel-s)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pagel-s)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
