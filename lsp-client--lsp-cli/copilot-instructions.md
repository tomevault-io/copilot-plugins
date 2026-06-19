## lsp-cli

> - Lint & format: `ruff check --fix && ruff format`

# AGENTS.md

## Development Commands

- Lint & format: `ruff check --fix && ruff format`
- Type checking: `ty check <dir_or_file>`
- Run tests: `uv run pytest`
- Sync latest deps: `uv sync --upgrade`

## Code Style Guidelines

- Python: 3.12+ required
- `attrs` for class definitions
- `anyio` and `asyncer` for async programming
- `typer` for cli, `rich` for printing

---
> Source: [lsp-client/lsp-cli](https://github.com/lsp-client/lsp-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
