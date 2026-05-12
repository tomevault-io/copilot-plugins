## markdown-preview-nvim

> Neovim plugin for live markdown preview in the browser. Pure Lua, no npm.

# markdown-preview.nvim

Neovim plugin for live markdown preview in the browser. Pure Lua, no npm.

## Project Structure

- `lua/markdown_preview/init.lua` — main plugin logic (server lifecycle, refresh, scroll sync, mermaid pre-rendering)
- `lua/markdown_preview/util.lua` — fs helpers, workspace resolution, asset resolution, browser open
- `lua/markdown_preview/ts.lua` — Tree-sitter mermaid extractor + regex fallback
- `lua/markdown_preview/lock.lua` — lock file management for takeover mode (cross-instance coordination)
- `lua/markdown_preview/remote.lua` — HTTP event injection for secondary instances (scroll sync)
- `plugin/markdown-preview.lua` — user commands (`:MarkdownPreview`, `:MarkdownPreviewStop`, `:MarkdownPreviewRefresh`)
- `assets/index.html` — browser preview app (CSS + JS, single file, ~1500 lines)

## Sibling Dependency

This plugin depends on [live-server.nvim](~/Repositories/Personal/Github/live-server.nvim) — a pure Lua HTTP server with SSE support. Both repos are owned by the same user and may be edited in the same session.

Key live-server APIs used:
- `server.start(cfg)` → returns instance with `.port`
- `server.stop(inst)`, `server.reload(inst, path)`, `server.send_event(inst, event, data)`
- `server.update_target(inst, root, index)`, `server.connected_client_count(inst)`
- `GET /__live/inject?event=<type>&data=<json>` — HTTP endpoint for external event injection

## Architecture

- Neovim writes markdown to `content.md` in a workspace directory
- live-server.nvim serves the workspace and pushes SSE events to the browser
- Browser uses markdown-it + morphdom for efficient DOM diffing
- SSE events: `reload` (content change), `scroll` (line-based sync)

### Instance Modes

- **takeover** (default): shared workspace + fixed port (8421). One browser tab across all Neovim instances. First instance is primary (runs server), others are secondary (write files only, server's fs_watch triggers reload).
- **multi**: per-instance server with OS-assigned port (port 0). Each instance gets its own browser tab.

## How to Test

1. Open a `.md` file in Neovim, run `:MarkdownPreview`
2. For takeover mode: open another `.md` in a separate Neovim instance, run `:MarkdownPreview` — same browser tab should update
3. For multi mode: set `instance_mode = "multi"` in setup, each instance opens a new tab
4. Test scroll sync by moving cursor — browser should follow
5. Test `:MarkdownPreviewStop` — server stops, lock file cleaned up (takeover primary)

## Conventions

- Lua patterns, not regex (no `{n,m}` quantifiers)
- `vim.loop` / `uv` for all async I/O
- No default keymaps — users map their own (per issue #4)
- No Co-Authored-By in commits
- Release titles: clean version numbers only (e.g., "v1.2.0")

---
> Source: [selimacerbas/markdown-preview.nvim](https://github.com/selimacerbas/markdown-preview.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
