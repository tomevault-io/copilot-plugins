## hummcode

> hummcode is a pip-installable Python coding agent. The entry point is

# AGENTS.md — hummcode project rules

## What this project is
hummcode is a pip-installable Python coding agent. The entry point is
`hummcode.core:main`. The source lives entirely in `src/hummcode/`.

## Architecture rules
- The core loop in `core.py` must stay thin. Business logic belongs in
  the relevant module (tools/, memory.py, llm.py), not in the loop.
- All tool schemas must be Pydantic BaseModel classes.
- All new tools must be registered in `tools/registry.py` — never
  called directly from `core.py`.
- `async/await` is required throughout. Never introduce blocking calls
  (no bare `input()`, no `time.sleep()`) inside the agent loop.

## Coding conventions
- Use type hints everywhere.
- Tool functions return strings — always. Errors are returned as
  strings (e.g. "Error: file not found"), never raised as exceptions
  that crash the loop.
- Pydantic schemas use `Field(..., description="...")` — the description
  is what the LLM reads to understand how to call the tool.

## Security rules
- NEVER run `edit_file` or `execute_bash` without going through
  `PermissionManager.check_permission()`.
- NEVER write secrets, API keys, or `.env` contents to any output.
- NEVER modify `.gitignore` without asking.

## What to never do
- Do not add new dependencies without updating `pyproject.toml`.
- Do not hardcode model names — always read from `DEFAULT_MODEL` env var.
- Do not add print() statements to the agent loop — use yield events.

## File structure
src/hummcode/
  core.py        — agent loop only, no business logic
  llm.py         — LiteLLM wrapper
  memory.py      — SessionTree, Node, compact()
  tools/
    registry.py  — all tool schemas and dispatch
    file_ops.py  — read_file, list_files, edit_file
    shell.py     — execute_bash
    oracle.py    — ask_oracle
    permissions.py — PermissionManager
  ui/
    tui.py       — Textual app, do not put agent logic here
    themes/      — CSS only, no Python

---
> Source: [0xchamin/hummcode](https://github.com/0xchamin/hummcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
