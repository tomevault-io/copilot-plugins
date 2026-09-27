## python

> These guidelines establish coding standards and best practices for Python code used in the backend of the Universal Token Launcher project. **Python 3.11** is required for all development.

# Python Guidelines and Rules for Backend

backend/** @guide 

These guidelines establish coding standards and best practices for Python code used in the backend of the Universal Token Launcher project. **Python 3.11** is required for all development.

---

## 1. Python Version

- **Requirement:** All backend code must run on **Python 3.11**.
- Use Python 3.11-specific features and type enhancements where applicable.

---

## 2. Code Style & Formatting

- **PEP8 Compliance:**  
  - Follow PEP8 guidelines strictly.
  - Use tools like Black for auto-formatting and flake8 for linting.

- **Indentation:**  
  - Use 4 spaces per indentation level (no tabs).

- **Line Length:**  
  - Limit lines to a maximum of 120 characters; longer lines should be wrapped appropriately.

- **Blank Lines & Spacing:**  
  - Separate top-level function and class definitions with two blank lines.
  - Use a single blank line between methods inside classes.
  - Maintain consistent spacing around operators and after commas.

- **Quotes:**  
  - Use single quotes (`'`) for strings, except when double quotes are needed for clarity.

- **Trailing Commas:**  
  - Use trailing commas in multiline data structures (lists, tuples, dictionaries) to simplify version control diffs.

---

## 3. Naming Conventions

- **Variables and Functions:** Use `snake_case` (e.g., `token_balance`, `deploy_contract()`).
- **Classes and Exceptions:** Use `PascalCase` (e.g., `TokenDeployer`, `DeploymentError`).
- **Constants:** Use `UPPER_CASE_WITH_UNDERSCORES` (e.g., `MAX_ENTRIES = 100`).

---

## 4. Module and File Organization

- Organize code into logical modules and packages following the project’s folder structure.
- Use descriptive filenames that reflect the module’s purpose (e.g., `auth.py`, `database.py`, `services/deployer.py`).
- Keep modules focused on a single responsibility.

---

## 5. Type Hints & Annotations

- **Type Annotations:**  
  - All functions and methods should include type hints for parameters and return types.
  - Leverage Python 3.11’s improved type hinting features for clarity.

- **Docstrings:**  
  - Write clear docstrings for all public functions, classes, and modules.
  - Adopt a consistent style (Google style, NumPy style, or reStructuredText).

---

## 6. Error Handling & Exceptions

- **Explicit Exceptions:**  
  - Catch and raise specific exceptions. Avoid using bare `except:` clauses.
  - Define custom exception classes when needed.

- **Logging Errors:**  
  - Use Python’s built-in `logging` module to log errors, warnings, and informational messages.
  - Include stack traces and contextual information in logs.

---

## 7. Dependency Management

- Use a virtual environment (e.g., `venv` or Poetry) to manage dependencies.
- List all dependencies in a `requirements.txt` file or a `pyproject.toml` if using Poetry.
- Regularly update dependencies and pin versions to ensure reproducible builds.

---

## 8. Testing Guidelines

- **Test Framework:**  
  - Use `pytest` for writing unit and integration tests.

- **Coverage:**  
  - Aim for at least 80% test coverage.
  - Organize tests in a dedicated `/tests` directory, mirroring the project structure.

- **Best Practices:**  
  - Write clear, isolated tests for each function or module.
  - Use fixtures and parameterization where appropriate.

---

## 9. Logging and Monitoring

- Use the standard `logging` module for application logs.
- Configure logging levels (DEBUG, INFO, WARNING, ERROR) appropriately.
- Ensure logs are written to both console and file (or external logging services if available).

---

## 10. Code Review and Documentation

- **Comments:**  
  - Write inline comments sparingly; code should be self-explanatory.
  - Use comments to explain non-obvious logic and complex algorithms.

- **Documentation:**  
  - Maintain comprehensive documentation for modules, classes, and functions.
  - Update README files and developer guides as the project evolves.

- **Code Reviews:**  
  - Ensure all code is reviewed by peers before merging.
  - Follow the project’s code review process to maintain quality and consistency.

---

## 11. Additional Best Practices

- **Modularity:**  
  - Design functions to perform a single task. If a function grows too long (preferably more than 50 lines), refactor it into smaller helper functions.
  
- **Security:**  
  - Avoid hardcoding credentials or sensitive information. Use environment variables instead.
  - Sanitize all inputs and validate external data rigorously.

- **Performance:**  
  - Optimize critical code paths but prioritize readability and maintainability.
  - Profile performance if you suspect a bottleneck.

- **Consistency:**  
  - Consistently apply these guidelines across all backend code to ensure a uniform codebase.

---
> Source: [CharlieMc0/universal-token-launcher](https://github.com/CharlieMc0/universal-token-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
