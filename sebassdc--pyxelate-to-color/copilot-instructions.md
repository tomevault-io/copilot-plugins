## pyxelate-to-color

> - All commands use `uv run <tool>`; no pip installs or builds needed

# AGENTS.md

## Build & Run

- All commands use `uv run <tool>`; no pip installs or builds needed
- Run app: `uv run fastapi dev index.py`

## Lint

uv run ruff .

## Tests

uv run pytest
uv run pytest tests/test_*.py::test_func

## Style Guidelines

- 4-space indent; max line length 88
- Imports: stdlib > 3rd-party > local (isort)
- snake_case for funcs/vars; CamelCase for classes
- Use f-strings; add type hints for public APIs
- Handle errors with exceptions/HTTPException; use logging
- Use pathlib for file paths

No Cursor/Copilot rules detected.

## Git Practices

- Do not run `git diff` on lockfiles.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebassdc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sebassdc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
