## good-ai

> - Apply separation of concerns (Single Responsibility Principle).


# Writing Code

## System Design
- Apply separation of concerns (Single Responsibility Principle).
- Use dependency injection.
- Ensure domain objects maintain type integrity through persistence.
- Reuse existing services, domain objects, DTO, adapters, ports, and utilities.
- Apply plugin/registry patterns instead of long conditional chains for pluggable components.
- Always handle errors gracefully and provide clear error messages.
- Use a single configuration system (e.g., pydantic settings, dotenv, or YAML).
- Never hard-code environment-specific paths, API keys, or constants.
- Do not use print statements for debugging.
- Ensure proper resource cleanup (close files, DB connections, sessions; prefer context managers).
- Make sure to properly wire the features you add (including during testing/bugfixes) into the rest of the codebase.
- Always search architecture docs @docs/developer/architecture for relevant and existing components and to understand proper wiring of the new features.

## Code Quality
- Add type hints to all function parameters and return values.
- Function return values must conform to their type hints.
- Avoid mutable default arguments (`[]`, `{}`); use `None` or immutable types instead.
- Use absolute imports.
- Use specific Exception clauses, not broad `except Exception`.
- Use descriptive names for functions, classes, and variables.
- Provide docstrings for all functions, classes, and modules (PEP 257).
- Treat warnings as errors during testing unless explicitly silenced in config.
- Always fix lint and type errors before proceeding to the next step.

## Tests Design
- Ensure you implement all required tests.
- The tests have check the correct behavior of the system, rather then try to match existing state.
- The aim is not only create reference for the future, but also identify problem points in the current implementation, which could be buggy or incomplete, and fix those problems throughout the codebase.
- Required test types:
  - Unit tests (mock external dependencies).
  - Integration tests (cross-module interactions).
  - Contract tests (enforce shared interfaces).
  - Consistency tests (validate uniform outputs).
  - End-to-end tests (validate full workflows).
  - Smoke tests (fast sanity checks after each build).
  - Implement tests to verify data type contracts between layers.
  - Add integration tests to check the proper wiring of the new features and their interaction with other services, ports, adapters, etc.
- Tests must not rely on log inspection for validation.
- During debugging, run only failing or new tests.
- Test names must describe behavior (e.g., `test_login_rejects_invalid_password`).
- If an import is missing, always search for the correct one.
- Never comment out code to fix errors.

## Linting and Testing Procedure
- Run `cat $filename | wc -l` to get file length of each modified python file and decompose files over 600 lines into smaller parts.
- Run `ruff check --fix` and fix errors.
- Run basedpyright and fix errors.
- Run pytest and fix errors.
- For gated tests, always run a gated version without requesting approval.
- Run and fix the tests and codebase till their pass rate is 100%.

## Documentation
- Make sure to properly document the features you add in @docs/developer/architecture.
- Documentation should be optimal from an agentic LLM use perspective.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyberbunny19) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
