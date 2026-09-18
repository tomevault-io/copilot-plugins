## automaya-mcp

> Read `vault/00 Home.md` for the map of the project, then the note that matches your task. Key rules:

# AutoMaya MCP: notes for Claude working in this repo

Read `vault/00 Home.md` for the map of the project, then the note that matches your task. Key rules:

- Plugin code (`maya_plugin/`) is stdlib + maya only; server code (`src/`) is Python 3.10+, FastMCP, httpx, pydantic.
- A new capability is three files (handler, tool module, test); the contract is `docs/DOMAIN_MODULE_CONTRACT.md`.
- Every mutating bridge command uses `mutates=True` so it gets undo rollback.
- Run `python3 -m pytest -q` and `ruff check src maya_plugin tests unreal` before finishing; regenerate `docs/TOOLS.md` with `scripts/gen_tool_catalogue.py` when tools change.
- No em dashes or en dashes in code, docs or messages. Casual, clear docstrings.
- The owner is a previs artist; when in doubt, favour what a production previs team would expect (real world units, cameras with proper sensors, shots in the sequencer).

---
> Source: [CallMeParams/automaya-mcp](https://github.com/CallMeParams/automaya-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
