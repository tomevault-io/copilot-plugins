## ghost-ship

> Update README.md when you find new info that would be useful for local setup or useful as general info about the project.

## General
Update README.md when you find new info that would be useful for local setup or useful as general info about the project.

CLI runs: enable uv venv with `source .venv/bin/activate && uv pip install -r requirements.txt` before running python files.

Prefer composition (with dependency injection) over inheritance where possible.
Use Protocol when an interface emerges. Protocols must not have props, just methods!

run `make lint` after you make changes

## Import Conventions
- No `__init__.py` files — always import from modules explicitly (e.g. `from context_providers.registry import build_registry`)
- All imports at the top of the file, never locally inside functions

---
> Source: [pavlos-io/ghost-ship](https://github.com/pavlos-io/ghost-ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
