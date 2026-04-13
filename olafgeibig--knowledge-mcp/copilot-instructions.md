## knowledge-mcp

> - **Elegance and Readability:** Strive for elegant and Pythonic code that is easy to understand and maintain.


# Coding

## Pythonic Practices

- **Elegance and Readability:** Strive for elegant and Pythonic code that is easy to understand and maintain.
- **PEP 8 Compliance:** Adhere to PEP 8 guidelines for code style, with Ruff as the primary linter and formatter.
- **Explicit over Implicit:** Favor explicit code that clearly communicates its intent over implicit, overly concise code.
- **Zen of Python:** Keep the Zen of Python in mind when making design decisions.

## Modular Design

- **Single Responsibility Principle:** Each module/file should have a well-defined, single responsibility.
- **Reusable Components:** Develop reusable functions and classes, favoring composition over inheritance. Keep functions and methods small and focused on a single task.
- **Package Structure:** Organize code into logical packages and modules.
- **Do not over-engineer solutions. Strive for simplicity and maintainability while still being efficient.**
- **Favor modularity, but avoid over-modularization.**

## Code Quality

- **Comprehensive Type Annotations:** All functions, methods, and class members must have type annotations, using the most specific types possible.
- **Detailed Docstrings:** All functions, methods, and classes must have Google-style docstrings, thoroughly explaining their purpose, parameters, return values, and any exceptions raised. Include usage examples where helpful.
- **Thorough Unit Testing:** Aim for high test coverage (90% or higher) using `pytest`. Test both common cases and edge cases.
- **Robust Exception Handling:** Use specific exception types, provide informative error messages, and handle exceptions gracefully. Implement custom exception classes when needed. Avoid bare `except` clauses.
- **Logging:** Employ the `logging` module judiciously to log important events, warnings, and errors.

## Other
- **Always consider the security implications of your code, especially when dealing with user inputs and external data.**
- Use list, dict, and set comprehensions when appropriate for concise and readable code.
- Prefer pathlib over os.path for file system operation
- Use docstrings for all public modules, functions, classes, and methods
- Don't write too many inline comments. 
- Use dataclasses for data containers when appropriate

# Technology Stack

- **Python:** Python 3.12 features and syntax
- **Dependency management:** uv
- **Code Formatting:** Ruff (replaces `black`, `isort`, `flake8`)
- **Testing:** Pytest
- **Documentation:** Google style docstring
- **Build system:** hatchling
  
## Using uv

- `uv add <dependency name>` to add dependencies
- `uv remove <dependency name>` to remove dependencies
- `uv pip` only if it is really necessary
- Create uv scripts for running scripts in pyproject.toml [project.scripts]

## Running Code

- Ensure activate the venv before spawning a new console session: `source .venv/bin/activate`
- Use uv scripts to run something

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/olafgeibig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
