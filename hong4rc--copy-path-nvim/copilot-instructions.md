## copy-path-nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

copy-path.nvim — a Neovim plugin (Lua only, no external dependencies) that copies buffer path variants to the system clipboard. Min Neovim 0.9.

## Architecture

The plugin follows standard Neovim plugin layout with lazy.nvim compatibility:

- `lua/copy-path/init.lua` — Public API and resolvers. Exports `setup()`, `copy(kind)`, `get(kind)`, `pick()`, `resolvers()`. Owns the 11 path resolvers and the `_copy()` helper that writes to `+` and `"` registers.
- `lua/copy-path/config.lua` — Default config, `resolve()` (deep-merge via `vim.tbl_deep_extend`), `validate()` (returns error list).
- `lua/copy-path/commands.lua` — Registers `:CopyPath*` user commands. Maps kind strings to command suffixes. Delegates to `copy_fn(kind)`.
- `lua/copy-path/keymaps.lua` — Registers normal/visual-mode keymaps and which-key groups. Delegates to `copy_fn(kind)`.
- `lua/copy-path/health.lua` — `:checkhealth copy-path` implementation.
- `lua/copy-path/git.lua` — GitHub/GitLab permalink generation. Resolves remote URL, branch/commit, and relative path.
- `lua/copy-path/tree.lua` — File tree integration (neo-tree, nvim-tree, snacks explorer). Extracts path from tree node under cursor.
- `lua/copy-path/picker.lua` — Built-in picker (Snacks, Telescope, fzf-lua, vim.ui.select fallback). Shows all kinds with current values.
- `plugin/copy-path.lua` — Guard file only (`vim.g.loaded_copy_path`). Does NOT call `setup()`.

**Two separate guards:** `vim.g.loaded_copy_path` (plugin file load) and `vim.g.copy_path_setup_done` (setup call). Both must be respected.

**Flow:** `setup(opts)` → resolve config → validate → register commands → register keymaps → register which-key → register picker keymap.

## Kinds

11 path kinds: `relative`, `full`, `filename`, `stem`, `extension`, `dir_full`, `dir_rel`, `line`, `line_full`, `github`, `github_line`. These are the canonical keys used in config, commands, keymaps, and resolvers.

## Default Keymaps (LazyVim-safe)

`<leader>fy`, `<leader>fY`, `<leader>fN`, `<leader>fl`, `<leader>fG` + `<leader>fP` (picker). These avoid conflicts with LazyVim core bindings (`<leader>ft` = Terminal, `<leader>fe` = Explorer, `<leader>fn` = New File).

## Testing

Tests use plenary.nvim. Run headless:
```bash
nvim --headless -u tests/minimal_init.lua \
  -c "PlenaryBustedDirectory tests/ { minimal_init = 'tests/minimal_init.lua' }"
```

## Linting

```bash
luacheck lua/
```

Config in `.luacheckrc`: `std = "luajit"`, globals = `{ "vim" }`.

## Demo

Demo assets live on the orphan `demo` branch (not in main). To regenerate:
```bash
git checkout demo && bash record.sh && git add demo.gif && git commit && git vanity ffffff && git push origin demo
```

## Local Development

The plugin is installed locally in LazyVim via:
```lua
-- ~/.config/nvim/lua/plugins/copy-path.lua
return { dir = "/Users/anhhong/project/neovim", event = "VeryLazy", opts = {} }
```
Changes are picked up on Neovim restart.

---
> Source: [hong4rc/copy-path.nvim](https://github.com/hong4rc/copy-path.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
