## projects

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

This is the **Godot MCP Pro plugin** — a Godot 4 editor plugin that exposes ~172 GDScript tools over WebSocket to an external Node.js MCP server. The plugin lives entirely in `addons/godot_mcp/`. The project root ("ProjectS") exists only to host and test the plugin inside the Godot editor.

## Architecture

```
External Node.js MCP server (ports 6505–6514)
        ↕  WebSocket (JSON-RPC 2.0)
addons/godot_mcp/websocket_server.gd   ← Godot acts as WS CLIENT
        ↕
addons/godot_mcp/command_router.gd     ← routes method → handler
        ↕
addons/godot_mcp/commands/*.gd         ← one file per domain
```

**Important:** The Godot plugin is the WebSocket *client*, not the server. It dials out to 10 Node.js MCP server instances on ports 6505–6514 (each Claude session gets its own port) and reconnects automatically every 3 seconds.

### Editor vs. Runtime split

- **Editor-side commands** (`commands/*.gd`) run inside the Godot editor process via `EditorInterface`. They are always available.
- **Runtime commands** (`mcp_game_inspector_service.gd`, `mcp_input_service.gd`, `mcp_screenshot_service.gd`) run inside the *game* process as autoloads. They communicate with the editor through temp files in `OS.get_user_data_dir()`:
  - `mcp_game_request` — editor writes a JSON command, game reads and deletes it
  - `mcp_game_response` — game writes the result, editor reads and deletes it
  - `mcp_screenshot_request` — triggers screenshot captures
  - `mcp_debugger_continue` — flag file that triggers auto-pressing the debugger Continue button

### Key files

| File | Role |
|---|---|
| `plugin.gd` | `EditorPlugin` entry point; manages lifecycle, autoload injection, dialog auto-dismiss |
| `websocket_server.gd` | WebSocket client; polls all 10 ports, dispatches JSON-RPC messages |
| `command_router.gd` | Loads all command modules, maps method names → callables, per-tool disable/enable |
| `commands/base_command.gd` | Base class with shared helpers (`success()`, `error_*()`, `require_string()`, `find_node_by_path()`, etc.) |
| `commands/*.gd` | Domain command modules (25 files); each implements `get_commands() -> Dictionary` |
| `mcp_game_inspector_service.gd` | Runtime autoload: scene tree, property inspection, frame capture, property monitoring, signal watching, move_to |
| `mcp_input_service.gd` | Runtime autoload: key/mouse/action simulation |
| `mcp_screenshot_service.gd` | Runtime autoload: screenshot capture via temp-file IPC |
| `utils/property_parser.gd` | Static helpers: `parse_value()` (string→Godot type), `serialize_value()` (Godot type→JSON) |
| `utils/node_utils.gd` | Static node traversal helpers |
| `ui/status_panel.gd` | Editor bottom-panel UI showing connection status and per-tool toggles |

## Adding a New Command

1. Pick the appropriate `commands/*.gd` file for the domain (or create a new one extending `base_command.gd`).
2. Add a method `func _cmd_my_tool(params: Dictionary) -> Dictionary`.
3. Return `success({...})` or one of the `error_*()` helpers.
4. Register it in `get_commands()`:
   ```gdscript
   func get_commands() -> Dictionary:
       return {
           "existing_tool": _cmd_existing_tool,
           "my_new_tool": _cmd_my_tool,   # add here
       }
   ```
5. If creating a new file, add it to the `command_classes` array in `command_router.gd`.
6. Use `validate_script` / `get_editor_errors` to check for syntax errors after editing.

### Command return conventions

```gdscript
# Success
return success({"key": "value"})

# Error variants
return error_not_found("Node", "Check the node path")
return error_invalid_params("Missing required param: node_path")
return error_no_scene()
return error_internal("Unexpected state")
```

### Accessing the editor from a command

All command nodes have `editor_plugin` set by the router. Use the base-class helpers:

```gdscript
get_editor()        # → EditorInterface
get_edited_root()   # → the currently open scene's root Node
get_undo_redo()     # → EditorUndoRedoManager
find_node_by_path("NodeName/Child")  # handles relative/absolute paths
```

## GDScript Conventions

- All plugin files use `@tool` so they run in the editor.
- Use explicit type annotations on `for` loop variables: `for item: String in arr` not `for item in arr`.
- Prefer `await command_router.execute(method, params)` pattern — commands can be async.
- Property values passed from MCP clients are strings; use `PropertyParser.parse_value()` or the base-class `require_string()` / `optional_*()` helpers to validate and convert.

## Testing Changes

There is no automated test suite. Testing workflow:
1. Edit GDScript in the Godot editor (or via `edit_script` MCP tool).
2. Use `validate_script` to check syntax.
3. Call `reload_project` (or toggle the plugin off/on in Project → Plugins) to pick up changes.
4. Use `get_editor_errors` and `get_output_log` to check for runtime errors.
5. For runtime changes, `play_scene` → exercise the command → `stop_scene`.

## Project Settings

- Physics engine: Jolt Physics (3D)
- Renderer: GL Compatibility (Windows: D3D12)
- Godot version: 4.6



Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

Tradeoff: These guidelines bias toward caution over speed. For trivial tasks, use judgment.

1. Think Before Coding
Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

State your assumptions explicitly. If uncertain, ask.
If multiple interpretations exist, present them - don't pick silently.
If a simpler approach exists, say so. Push back when warranted.
If something is unclear, stop. Name what's confusing. Ask.
2. Simplicity First
Minimum code that solves the problem. Nothing speculative.

No features beyond what was asked.
No abstractions for single-use code.
No "flexibility" or "configurability" that wasn't requested.
No error handling for impossible scenarios.
If you write 200 lines and it could be 50, rewrite it.
Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

3. Surgical Changes
Touch only what you must. Clean up only your own mess.

When editing existing code:

Don't "improve" adjacent code, comments, or formatting.
Don't refactor things that aren't broken.
Match existing style, even if you'd do it differently.
If you notice unrelated dead code, mention it - don't delete it.
When your changes create orphans:

Remove imports/variables/functions that YOUR changes made unused.
Don't remove pre-existing dead code unless asked.
The test: Every changed line should trace directly to the user's request.

4. Goal-Driven Execution
Define success criteria. Loop until verified.

Transform tasks into verifiable goals:

"Add validation" → "Write tests for invalid inputs, then make them pass"
"Fix the bug" → "Write a test that reproduces it, then make it pass"
"Refactor X" → "Ensure tests pass before and after"
For multi-step tasks, state a brief plan:

1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---
> Source: [kio972/ProjectS](https://github.com/kio972/ProjectS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
