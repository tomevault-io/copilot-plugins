## musubi

> These conventions apply to all code produced in this repository.

# GitHub Copilot — Team Conventions

These conventions apply to all code produced in this repository.

## Language and Runtime

- Python 3.11+ only. No Python 2 compatibility.
- Use `pyproject.toml` for project configuration. No `setup.py` or `setup.cfg`.
- Manage dependencies with `uv` or `pip`. Pin versions in `pyproject.toml`.

## Code Style

- Follow PEP 8. Line length: 100 characters.
- Use `ruff` for linting and formatting. Config lives in `pyproject.toml`.
- Use `mypy` for static type checking. Strict mode enabled.
- All public functions, classes, and methods must have type annotations.
- No `# type: ignore` without an explanatory comment.

## Type Hints

```python
# correct
def process(items: list[str]) -> dict[str, int]: ...

# wrong — missing annotations
def process(items, data): ...
```

## Docstrings

- Public modules, classes, and functions get a one-line docstring.
- No multi-paragraph docstrings. One sentence is almost always enough.
- Format: Google style.

## Error Handling

- Catch specific exceptions, not bare `except:` or `except Exception:`.
- Always re-raise after logging at unexpected failure points.
- Use custom exception classes for domain errors.

## Security

- Never hardcode secrets, credentials, API keys, or tokens.
- Use environment variables for configuration. Access via `os.environ` with explicit fallback or raise.
- Validate all input at system boundaries. Trust nothing from external sources.
- See `.github/instructions/universal/security.instructions.md` for full rules.

## Git Conventions

- Branch names: `type/short-description` (e.g., `feat/login-endpoint`, `fix/null-check`)
- Commit messages: imperative mood, under 72 characters.
  - Format: `type(scope): description`
  - Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- One logical change per commit.
- No merge commits on feature branches — rebase.

## Testing

- All new code must have pytest tests.
- Test file location: `tests/test_{module}.py` mirroring the source tree.
- Test function names: `test_{what}_{condition}_{expected_result}`.
- Minimum coverage: 80% per module.
- Tests must pass before a PR is merged.

## MCP Tool Calls (agent code)

- Agents interact with session state **only** via MCP tool calls.
- Never construct prompts inline — context is assembled by `context_builder.py`.
- Pass `agent_name` on every `musubi_read_stage` call.
- Schema: see each `.agent.md` Output Contract section.

## File Organization

```
musubi/
    server.py        ← MCP entry point only
    state.py         ← session state
    context_builder.py
    verifier.py
    executor.py
    correction_loop.py
    skill_loader.py
    memory/
    storage/
    tests/
```

No business logic in `server.py`. It only routes tool calls.

---
> Source: [Eurus7895/Musubi](https://github.com/Eurus7895/Musubi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
