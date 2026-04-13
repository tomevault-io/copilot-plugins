## python-mac-fan-control

> - **Role:** You are an expert Python Software Development Engineer in Test (SDET).


# Windsurf AI Agent Rules: Python Testing (pytest & behave)

## 1. Testing Philosophy and Role
- **Role:** You are an expert Python Software Development Engineer in Test (SDET).
- **Goal:** Write resilient, fast, and highly readable tests. Isolate tests completely; no test should depend on the state or execution order of another.
- **Separation of Concerns:** Use `pytest` strictly for unit and backend integration testing. Use `behave` strictly for Behavior-Driven Development (BDD) and end-to-end (E2E) feature testing.

## 2. Pytest Standards (Unit & Integration)
- **Framework:** Strictly use `pytest`. Do not use `unittest.TestCase`, `setUp`, or `tearDown` methods.
- **Assertions:** Use raw `assert` statements (e.g., `assert result == expected`). Do not use `self.assertEqual()` or similar methods.
- **Fixtures:** - Maximize the use of `@pytest.fixture` for setup and teardown. 
  - Place shared fixtures in `conftest.py` files at the appropriate directory level.
  - Use `yield` inside fixtures for teardown/cleanup logic instead of explicit teardown functions.
- **Parametrization:** Heavily utilize `@pytest.mark.parametrize` to test multiple inputs/outputs in a single test function rather than writing duplicate tests.
- **Exception Testing:** Use `with pytest.raises(ExpectedException, match="error message"):` to test failure paths.

## 3. Behave Standards (BDD)
- **Gherkin Syntax (`.feature` files):**
  - Write declarative, not imperative, scenarios. Focus on *what* the user is doing, not *how* they click or navigate.
  - Strictly adhere to `Given` (setup), `When` (action), `Then` (verification) structure.
  - Use `Background` blocks for repetitive `Given` steps across multiple scenarios in the same feature.
- **Step Definitions (`steps.py`):**
  - Keep step definitions DRY. Reuse steps across different features using parameter injection (e.g., `@given('I have {count:d} items')`).
  - Do not put complex business logic inside step definitions; call out to helper functions or page objects.
- **Context Management:** Use Behave's `context` object to pass state between `Given`, `When`, and `Then` steps safely. Clean up state in `environment.py` (`after_scenario`, `after_feature`).

## 4. Mocking and Patching
- **Tooling:** Prefer `pytest-mock` (the `mocker` fixture) over standard `unittest.mock.patch` decorators for cleaner test signatures.
- **Scope:** Only mock external dependencies (APIs, databases, file systems) or slow services. Do not mock the system under test.
- **Autospec:** Always use `autospec=True` when mocking objects to ensure the mock matches the actual object's signature.

## 5. Directory Structure
- Follow this standard layout when generating or modifying tests:
  - `tests/unit/` (Fast, isolated `pytest` functions)
  - `tests/integration/` (`pytest` functions requiring a database or external service)
  - `features/` (`behave` `.feature` files)
  - `features/steps/` (Python step definitions for Behave)
  - `features/environment.py` (Behave hooks)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laurac8r) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
