## sdd1

> This document contains all development rules and guidelines for this project, applicable to all AI agents (Claude, Cursor, Codex, Gemini, OpenCode, etc.).


## 1. Core Principles

- **Small tasks, one at a time**: Always work in baby steps, one at a time. Never go forward more than one step.
- **Test-Driven Development**: Start with failing tests for any new functionality (TDD), according to the task details.
- **Type Safety**: All Python code must use type hints (PEP 484). Use mypy or pyright for static type validation.
- **Clear Naming**: Use clear, descriptive names for all variables and functions following PEP 8 conventions.
- **Incremental Changes**: Prefer incremental, focused changes over large, complex modifications.
- **Question Assumptions**: Always question assumptions and inferences.
- **Pattern Detection**: Detect and highlight repeated code patterns.

## 2. Language Standards
- **English preferred**: All **new** technical artifacts should use English, including:
    - Code (variables, functions, classes, comments, error messages, log messages)
    - Documentation (README, guides, API docs)
    - Task files and change proposals (titles, descriptions, comments)
    - Data schemas and database names
    - Configuration files and scripts
    - Git commit messages
    - Test names and descriptions
- **Spanish exceptions (existing code)**: The production codebase contains Spanish identifiers, log messages, and domain-specific terms inherited from earlier development. These are acceptable in existing code and should **not** be renamed solely for language consistency. When modifying existing Spanish-named code, keep surrounding naming consistent with its module. New modules and files should follow the English convention above.

## 3. Specific standards

For detailed standards and guidelines specific to different areas of the project, refer to:

- [Backend Standards](./backend-standards.mdc) - Python development, Flask API, database patterns, testing with pytest, and backend best practices
- [Frontend Standards](./frontend-standards.mdc) - React dashboards, Streamlit dashboards, UI guidelines, and frontend architecture
- [Documentation Standards](./documentation-standards.mdc) - Technical documentation structure, formatting, and maintenance guidelines, including AI standards like this document

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lsilva5455) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
