## astronvim-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an **AstroNvim v5+** user configuration, a Neovim distribution built on lazy.nvim. The configuration is tailored primarily for **R language development** with secondary support for Lua.

## Commands

```bash
stylua lua/          # Format all Lua files
selene lua/          # Lint all Lua files (selene must be installed separately; not currently on PATH)
nvim --headless "+Lazy! sync" +qa   # Sync plugins non-interactively
```

## Architecture

### Load Order

`init.lua` bootstraps lazy.nvim, then calls `lua/lazy_setup.lua` which imports specs in this order:

1. **AstroNvim base** (`astronvim.plugins`) — core framework
2. **AstroCommunity** (`lua/community.lua`) — community language packs and colorschemes
3. **User plugins** (`lua/plugins/`) — custom overrides and additions

lazy.nvim merges specs that share the same plugin name, so a user plugin file can override community or base defaults by targeting the same `"Owner/Repo"` string.

`lua/polish.lua` runs last but is currently **disabled** (early return on line 1).

### Plugin Spec Pattern

All plugin configs follow the lazy.nvim spec format:
```lua
return {
  "Plugin/Name",
  opts = { ... },  -- merged into setup() call
  config = function(_, opts) ... end,  -- optional: replaces default setup
}
```

### Key Files

| File | Purpose |
|------|---------|
| `lua/lazy_setup.lua` | Main lazy.nvim config; sets leader keys and import order |
| `lua/community.lua` | AstroCommunity imports (Lua pack, colorschemes) |
| `lua/plugins/astrocore.lua` | Vim options, keymaps, diagnostics, features |
| `lua/plugins/astrolsp.lua` | LSP config: format-on-save, codelens, semantic tokens |
| `lua/plugins/r-nvim.lua` | R.nvim + neotest-testthat |
| `lua/plugins/user.lua` | Snacks.nvim (dashboard, picker, explorer, etc.), LuaSnip, autopairs |
| `lua/plugins/none-ls.lua` | none-ls with `write_good` diagnostics |
| `lua/plugins/mason.lua` | Mason auto-install: lua-language-server, stylua, debugpy, tree-sitter-cli |

## Code Style

- **Formatter**: StyLua — 120 column width, 2-space indent, no call parentheses (see `.stylua.toml`)
- **Linter**: Selene with `std = "neovim"` and several rules relaxed (see `selene.toml`)
- Format Lua before committing: `stylua lua/`

## Key Customizations

- **Leader**: Space (`mapleader = " "`)
- **Local leader**: Backslash (`maplocalleader = "\\"`) — keeps R.nvim's `\rf`, `\l`, etc. separate from AstroNvim mappings
- `c` key disabled in normal mode
- `]b`/`[b` for buffer navigation
- `gD` for LSP declaration
- Format on save enabled (1000ms timeout)
- Codelens enabled with auto-refresh on `InsertLeave` and `BufEnter`
- Diagnostics: virtual text on, virtual lines off
- Line wrap on, absolute line numbers (no relative)
- Large file threshold: 256KB or 10,000 lines

## Language Support

- **R**: R.nvim (main branch, not lazy-loaded) with neotest-testthat. PDF viewer disabled (`pdfviewer = ""`); uses `vim.ui.open` for PDF opening instead.
- **Lua**: AstroCommunity Lua pack with lua-language-server via Mason
- **Prose**: `write_good` diagnostics via none-ls
- **Treesitter parsers**: lua, vim, markdown, markdown_inline, r, rnoweb, yaml, latex, csv

## Colorschemes

Active: `astrodark`. Available: catppuccin (with extensive integrations configured), solarized-osaka, monokai-pro, NeoSolarized.

---
> Source: [bowers-illinois-edu/astronvim-config](https://github.com/bowers-illinois-edu/astronvim-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
