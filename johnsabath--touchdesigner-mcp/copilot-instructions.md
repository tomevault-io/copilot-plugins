## touchdesigner-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A generic MCP server (`scripts/td_mcp_server.py`) that runs inside TouchDesigner via a Web Server DAT on port 9988. It implements MCP Streamable HTTP (JSON-RPC over POST to `/mcp`) with no external dependencies beyond TD itself.

## Development Workflow

After modifying `scripts/td_mcp_server.py`, reload into TD and rebuild the `.tox`:

```
td_run: exec(open(project.folder + '/scripts/rebuild_tox.py').read())
```

This syncs the script into the handler DAT and re-exports `td_mcp_server.tox`. The MCP client may need a reconnect (`/mcp`) to pick up new tools.

## Architecture

Single-file server (`scripts/td_mcp_server.py`) attached to a Web Server DAT. The entry point is `onHTTPRequest()` which routes to `_handle_request()` → `_handle_one()` → individual `handle_*` functions via `TOOL_HANDLERS` dispatch dict.

Key subsystems:
- **Tool definitions**: `TOOLS` list (JSON Schema) at top of file, handlers named `handle_<tool_name>`
- **Frame capture**: Synchronous capture via `_capture_frame()` — advances the timeline and force-cooks the operator chain each frame. Used by the `observe` tool.
- **APNG encoder**: `_encode_apng()` — pure Python animated PNG encoder (no PIL/ffmpeg needed)
- **Root discovery**: `_get_root()` uses `me.parent()` so the server works regardless of where it's placed in the operator hierarchy

## Tools (13)

`run`, `inspect`, `set`, `create`, `wire`, `observe`, `render`, `read`, `write`, `edit`, `list`, `docs`, `map`

**Use `docs` before creating operators** to look up correct parameter names, menu values, and type constants. TD parameter names are often abbreviated (e.g. `rough` not `roughness`, `wavetype` not `type`, `frequency` not `freq`). Use `docs` with `type='list_types'` to find type constants like `lfoCHOP`, `noiseTOP`, etc.

**Use `map` before creating operators** to see the current network layout with `@(x,y)` coordinates, then place new operators nearby using the required `nodeX`/`nodeY` params on `create`.

### Adding a new tool

1. Add its JSON Schema to the `TOOLS` list
2. Write a `handle_<name>(args)` function
3. Add it to `TOOL_HANDLERS` dict
4. Update tool counts/lists in: this file, `../CLAUDE.md`, `README.md`

## Project Files

- `scripts/td_mcp_server.py` — the MCP server source (synced to a TextDAT in TD)
- `scripts/rebuild_tox.py` — reloads server script into TD and re-exports the `.tox`
- `td_mcp_server.tox` — pre-wired component (Web Server DAT + callback script), drag-and-drop install
- `.mcp.json` — Claude Code MCP client config pointing to `localhost:9988/mcp`
- `example.toe` — self-contained demo project (not tracked in git, binary)

## Working with TouchDesigner

- Operator paths are absolute (e.g. `/project1/out`). The root container is auto-discovered via `me.parent()`.
- `.toe` files are binary — can't be diffed or merged. All `.toe` files are gitignored.
- TD's Python environment is embedded (no pip). Only stdlib + numpy are available.
- To set expressions on parameters, use `par.name.expr = "..."` — assigning a string directly to a numeric par will error.
- After writing a GLSL shader, call `op.par.loaduniformnames.pulse()` to auto-detect uniforms.

---
> Source: [johnsabath/touchdesigner-mcp](https://github.com/johnsabath/touchdesigner-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
