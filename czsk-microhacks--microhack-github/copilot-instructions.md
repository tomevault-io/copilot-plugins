## microhack-github

> Provide a clear, single reference for implementing, extending, and maintaining AI agents and related services in this repository. This project targets **simplicity** to facilitate learning.

# Agent Development Guidelines

## 1. Purpose

Provide a clear, single reference for implementing, extending, and maintaining AI agents and related services in this repository. This project targets **simplicity** to facilitate learning.

## 2. Core Principles

1.  **Simplicity First**: Do not overcomplicate. Avoid premature abstractions. Keep code easy to read and understand for learners.
2.  **Self-Documenting Code**: Use docstrings, not progress/status comments.
3.  **Small & Cohesive**: Minimize surface area. Small, cohesive modules are better than large monoliths.
4.  **Explicit is Better than Implicit**: Be explicit about data contracts, configuration, and side effects.
5.  **Disposable Experiments**: Make cheap experiments disposable (prefixed `adhoc_`), not permanent.

## 3. Project-Wide Conventions

### 3.1 Documentation
*   **Docstrings**: Primary documentation channel inside code. Revise them whenever code changes behavior or signature.
*   **Comments**: Only add code comments for non-obvious logic or critical nuances. Never for progress logs or "previous implementation" commentary.
*   **Implementation Log**: Update `specs/IMPLEMENTATION_LOG.md` with meaningful architectural or technical decisions when a feature is completed.
*   **Common Errors**: Add confirmed recurring pitfalls to `specs/COMMON_ERRORS.md` (after user confirmation).
*   **README**: Each component/service keeps concise run & test instructions in its local `README.md`.

### 3.2 Refactoring & Improvements
Opportunistic simplifications are encouraged.
*   **Low-risk**: Perform obviously beneficial cleanups directly (pure simplification, dead code removal).
*   **Architectural**: For broader shifts, surface a brief rationale in chat before proceeding.

### 3.3 Experiments & Troubleshooting
*   Use `adhoc_` prefix for temporary files or scripts used for debugging or experimentation.
*   Delete these files once the insight is converted into real tests or code.

### 3.4 Technology Stack
*   **Language**: Python 3.11+
*   **Package Management**: `pip` with `requirements.txt`
*   **API Framework**: FastAPI
*   **Infrastructure as Code**: Bicep

## 4. Service Guidelines (Python)

### 4.1 Structure & Modeling
*   **Flat Structure**: Use a flat structure for microservices to reduce complexity.
    ```
    .
    ├── main.py           # Application entry point and route definitions
    ├── models.py         # Pydantic models and schemas
    ├── database.py       # Database service layer and connection logic
    ├── config.py         # Configuration and environment variable handling
    ├── requirements.txt  # Python dependencies
    ├── start.sh          # Startup and setup script
    ├── Dockerfile        # Container definition
    └── tests/            # Test files (e.g., test_main.py)
    ```
*   **Models**: `models.py` contains Pydantic schemas.
*   **Database**: `database.py` encapsulates all database logic (connection, CRUD, queries).
*   **Config**: `config.py` manages environment variables and application settings.

### 4.2 Documentation & Style
*   **PEP 8**: Follow PEP 8. Max line length 100 characters.
*   **Naming**: `snake_case` for functions/variables, `PascalCase` for classes.
*   **Type Hints**: Use type hints everywhere.
*   **Docstrings**: Every public function/class must have a docstring specifying purpose, parameters, return value(s), and exceptions.

### 4.3 Logging
*   **Library**: Use the standard `logging` library.
*   **No Print**: Do not use `print` statements in production code.
*   **Levels**: INFO for lifecycle events, DEBUG for diagnostics, WARNING for recoverable anomalies, ERROR for failures.

### 4.4 Testing
*   **Framework**: `pytest`.
*   **Unit Tests**: Keep them fast and free of live IO; mock repositories/interfaces.
*   **Integration Tests**: Provide integration tests for persistence layers and API contracts.
*   **Naming**: Descriptive test names (`test_<function>_<behavior>`).

## 5. Infrastructure as Code (Bicep)

### 5.1 Project Structure
*   **Modules**: Break down complex deployments into smaller, reusable modules.
*   **Main Entry Point**: Use a `main.bicep` file to orchestrate the deployment.
*   **Parameters**: Use `.bicepparam` or JSON parameter files.

### 5.2 Coding Conventions
*   **Naming**: `lowerCamelCase` for symbolic names. Use clear, descriptive names for parameters.
*   **Decorators**: Always provide `@description()` for parameters. Use `@secure()` for secrets.
*   **Symbolic References**: Use symbolic names to reference other resources (e.g., `storageAccount.id`) to create implicit dependencies.
*   **Existing Resources**: Use the `existing` keyword to reference resources not deployed in the current file.

## 6. Reinforced Documentation & Logging Rules

1.  **Implementation Log Boundaries**: Implementation progress belongs in `specs/IMPLEMENTATION_LOG.md`.
2.  **Common Errors Workflow**: Add to `specs/COMMON_ERRORS.md` only after confirmation.
3.  **No Progress Comments**: Ban inline comments like "// updated previous logic".

## 7. Ad-Hoc / Disposable Artifacts

| Type | Naming Pattern | Purpose | Lifecycle |
|------|----------------|---------|-----------|
| Scratch test script | `adhoc_test_*` or `adhoc_*` | Quick reproduction / isolate behavior | Delete after converting insight into real tests/code |
| Documentation draft | `ADHOC_*.md` | Staging ground for large doc refactor | Merge content into canonical doc then delete |

## 8. Change Control & Communication

1.  **Major Changes**: Summarize intent, risk, and alternatives in chat for approval before architectural changes.
2.  **Post-Implementation**: Update relevant docstrings and `specs/IMPLEMENTATION_LOG.md`.
3.  **Systemic Flaws**: Propose remediation path; avoid broad speculative refactors without confirmation.

## 9. Quick Reference Checklist

- [ ] Code follows PEP 8 and project structure (flat).
- [ ] Public functions/classes have docstrings.
- [ ] Type hints are used everywhere.
- [ ] No `print` statements; used `logging`.
- [ ] Unit tests created/updated with `pytest`.
- [ ] Bicep code uses symbolic names and descriptions.
- [ ] Simplicity maintained; no premature abstractions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CZSK-MicroHacks)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CZSK-MicroHacks)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
