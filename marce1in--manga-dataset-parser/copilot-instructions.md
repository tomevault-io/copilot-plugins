## manga-dataset-parser

> - Functions: 4-20 lines. Split if longer.

## Code style

- Functions: 4-20 lines. Split if longer.
- Files: under 500 lines. Split by responsibility.
- One thing per function, one responsibility per module (SRP).
- Names: specific and unique. Avoid `data`, `handler`, `manager`, `utils`.
  Prefer names that return <5 grep hits in the codebase.
- Types: explicit. No `Any` unless the boundary is genuinely dynamic.
  No untyped functions, no untyped parameters, no untyped returns.
- Prefer `dataclass`, `Protocol`, `TypedDict`, `NewType`, `Enum`, and `Literal`
  over loose dictionaries and stringly-typed values.
- No code duplication. Extract shared logic into a function/module.
- Early returns over nested ifs. Max 2 levels of indentation.
- Exception messages must include the offending value and expected shape.
- No framework-style globals, service containers, magic registries, or hidden app state.

## Comments

- Keep existing comments. Don't strip them on refactor — they carry intent
  and provenance.
- Write WHY, not WHAT. Skip `# increment counter` above `counter += 1`.
- Docstrings on public functions: intent + one usage example.
- Reference issue numbers / commit SHAs when a line exists because of a
  specific bug or upstream constraint.

## Tests

- Tests run with a single command: `uv run pytest`.
- Every new function gets a test. Bug fixes get a regression test.
- Mock external I/O (API, DB, filesystem, clock, env vars, subprocesses)
  with named fake classes, not inline stubs.
- Tests must be F.I.R.S.T: fast, independent, repeatable,
  self-validating, timely.
- Prefer plain `pytest` functions over class-based test suites unless
  shared state is required.
- Test behavior, not private implementation details.
- Code should pass all checks:
  `uv run ruff format --check .`
  `uv run ruff check .`
  `uv run mypy .`
  `uv run pytest`

## Dependencies

- Use `uv` for dependency management and command execution.
- Add runtime dependencies with `uv add <package>`.
- Add development dependencies with `uv add --dev <package>`.
- Remove dependencies with `uv remove <package>`.
- Sync the environment with `uv sync`.
- Run project-local tools with `uv run <command>`.
- Run one-off tools with `uvx <tool>`.
- Commit both `pyproject.toml` and `uv.lock`.
- Do not use `pip`, `pip-tools`, `poetry`, `pipenv`, or ad-hoc virtualenv
  commands in this project.
- Do not commit `.venv`.
- Inject dependencies through constructor/parameter, not global/import.
- Wrap third-party libs behind a thin interface owned by this project.
- Keep third-party imports at the boundary layer when possible.
- Do not leak third-party response objects into domain code.
- Prefer the standard library unless a dependency clearly reduces complexity.

## Structure

- Frameworkless means explicit composition. No hidden lifecycle, no service
  container, no magic discovery.
- Prefer small focused modules over god files.
- Suggested structure:
  `tests/` for unit and integration tests.
  `docs/` for structural and operational documentation.
- Avoid circular imports. If they appear, the module boundaries are wrong.
- Use `__init__.py` for package identity, not side effects or broad re-exports.

## Configuration

- Configuration is explicit and typed.
- Read environment variables only in one boundary module.
- Convert raw strings into typed config before passing them into application code.
- Fail fast on missing or malformed configuration.
- Error messages must name the missing variable and expected format.
- Do not access `os.environ` outside config loading code.

## CLI / Entrypoints

- Run CLI commands through `uv`: `uv run python -m project_name`.
- CLI output intended for users is plain text.
- Debug and observability logs are structured JSON.
- Exit codes are intentional:
  `0` success.
  `1` expected operational failure.
  `2` invalid user input.
  `>2` unexpected system failure.
- Keep `if __name__ == "__main__"` blocks thin.
- Entrypoints should parse input, call application code, render output, and exit.

## Docs

- Always update `/docs` with structural info when inline comments are not enough.
- Be objective in `/docs` files.
- Never make a docs file bigger than 100 lines. Break into more files if needed.
- Document module boundaries, external dependencies, configuration, commands,
  operational assumptions, and non-obvious tradeoffs.

## Formatting and linting

- Ruff is the only formatter and linter.
- Format code with `uv run ruff format .`.
- Fix lint issues with `uv run ruff check . --fix`.
- Check formatting with `uv run ruff format --check .`.
- Check linting with `uv run ruff check .`.
- Do not use Black, isort, Flake8, autopep8, yapf, or custom formatting scripts.
- Import order is managed by Ruff.
- Line length, quote style, and indentation are project settings in `pyproject.toml`.
- Don't discuss style beyond Ruff. If Ruff and personal preference disagree,
  Ruff wins.
- Do not add `# noqa` unless the rule is wrong for that case, the reason is
  written next to the suppression, and the suppression is as narrow as possible.

## Type checking

- Mypy is required.
- Run it with `uv run mypy .`.
- Domain and application code should be typed strictly.
- Untyped third-party data must be converted into project-owned typed values
  before entering domain code.
- Do not use `# type: ignore` unless the reason is written next to the ignore,
  the ignore uses a specific error code, and the code touches an unavoidable
  dynamic boundary.

## Logging

- Structured JSON when logging for debugging / observability.
- Plain text only for user-facing CLI output.
- Logs must include enough context to diagnose the event.
- Do not log secrets, tokens, credentials, raw personal data, or full request bodies.
- Prefer stable event names over prose messages.
- Include correlation/request IDs when available.

## Error handling

- Raise specific exceptions, not bare `Exception`.
- Do not swallow exceptions silently.
- Translate low-level exceptions at boundaries into project-owned exceptions.
- Preserve traceback context with `raise ... from err`.
- Validation errors must include the offending value and expected shape.
- User-facing errors should be concise and actionable.

## Python rules

- Target the Python version declared in `pyproject.toml`.
- Use `uv.lock` as the source of truth for resolved dependencies.
- Use `pathlib.Path`, not raw string path manipulation.
- Use timezone-aware `datetime` values.
- Use `subprocess.run(..., check=True)` with explicit arguments.
- Avoid mutable default arguments.
- Avoid module import side effects.
- Avoid monkeypatching production code.
- Prefer immutable value objects for domain data.
- Prefer generators for streaming data, lists for concrete collections.
- Do not use broad `except:` blocks.
- Do not use wildcard imports.

---
> Source: [Marce1in/manga-dataset-parser](https://github.com/Marce1in/manga-dataset-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
