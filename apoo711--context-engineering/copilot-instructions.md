## context-engineering

> _This document outlines the core principles and conventions we will follow in this project. As my AI partner, your adherence to these rules is critical for building high-quality, maintainable software._

# The Developer's Pact: Our Rules of Engagement
_This document outlines the core principles and conventions we will follow in this project. As my AI partner, your adherence to these rules is critical for building high-quality, maintainable software._

### 🏛️ Principle 1: Architecture & Structure
- **Modularity is Key:** No single file should exceed 500 lines. If it grows too large, your first step is to propose a refactoring plan to break it into smaller, logical modules.

- **Consistent Organization:** We group files by feature. For example, a new `user` feature would have its logic in `src/users/`, its API routes in `src/api/routes/users.py`, and its tests in `tests/users/`.

- **Clean Imports:** Use absolute imports for clarity (e.g., `from src.utils import helpers`). Avoid circular dependencies.

- **Environment First:** All sensitive keys, API endpoints, or configuration variables must be managed through a `.env` file and loaded using `python-dotenv`. Never hardcode them.

### ✅ Principle 2: Quality & Reliability
- **Test Everything That Matters:** Every new function, class, or API endpoint must be accompanied by unit tests in the tests/ directory.

- **The Test Triad:** For each feature, provide at least three tests:

  1. A "happy path" test for expected behavior.

  2. An "edge case" test for unusual but valid inputs.

  3. A "failure case" test for expected errors or invalid inputs.

- **Docstrings are Non-Negotiable:** Every function must have a Google-style docstring explaining its purpose, arguments (`Args:`), and return value (`Returns:`).

### ✍️ Principle 3: Code & Style
- **Follow the Standards:** All Python code must be formatted with `black` and adhere to `PEP8` guidelines.

- **Type Safety:** Use type hints for all function signatures and variables. We use `mypy` to enforce this.

- **Data Certainty:** Use `pydantic` for all data validation, especially for API request and response models. This is our single source of truth for data shapes.

### 🧠 Principle 4: Your Behavior as an AI
- **Clarify, Don't Assume:** If a requirement is ambiguous or context is missing, your first action is to ask for clarification.

- **No Hallucinations:** Do not invent libraries, functions, or file paths. If you need a tool you don't have, state what you need and why.

- **Plan Before You Code:** For any non-trivial task, first outline your implementation plan in a list or with pseudocode. We will approve it before you write the final code.

- **Explain the "Why":** For complex or non-obvious blocks of code, add a `# WHY:` comment explaining the reasoning behind the implementation choice.

---
> Source: [Apoo711/Context-Engineering](https://github.com/Apoo711/Context-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
