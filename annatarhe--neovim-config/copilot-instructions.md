## neovim-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal Neovim configuration using lazy.nvim as the plugin manager. The configuration is written entirely in Lua.

## Architecture

```
init.lua                    # Entry point - loads config modules in order
lua/
├── config/
│   ├── options.lua         # vim.opt settings (indent, UI, behavior)
│   ├── keymaps.lua         # Global key mappings (leader = ";")
│   ├── autocmds.lua        # Autocommands (yank highlight, filetype detection)
│   └── lazy.lua            # lazy.nvim bootstrap and plugin loader
└── plugins/                # Each file returns a table of plugin specs
    ├── colorscheme.lua     # onedark (primary), dracula
    ├── ui.lua              # lualine, neo-tree, which-key, indent-blankline
    ├── editor.lua          # Comment, autopairs, surround, visual-multi
    ├── git.lua             # gitsigns, fugitive
    ├── telescope.lua       # Fuzzy finder (replaces fzf)
    ├── treesitter.lua      # Syntax highlighting
    ├── lsp.lua             # nvim-lspconfig + mason for LSP management
    ├── cmp.lua             # nvim-cmp completion with LuaSnip
    ├── formatting.lua      # conform.nvim (format on save), nvim-lint
    └── languages.lua       # vim-go, emmet, graphql, wakatime
```

## Key Conventions

- **Leader key**: `;` (semicolon)
- **Plugin specs**: Use lazy.nvim format with lazy-loading via `event`, `cmd`, `ft`, or `keys`
- **LSP servers**: Managed by mason.nvim, configured in `lsp.lua`
- **Formatting**: Handled by conform.nvim (not LSP), runs on save

## Testing Configuration

Open Neovim and run:
```vim
:checkhealth          " Verify all plugins and dependencies
:Lazy                 " Check plugin status
:Mason                " Check LSP server installation
:LspInfo              " Check active LSP clients
```

## Adding New Plugins

Create or edit a file in `lua/plugins/`:
```lua
return {
  {
    "author/plugin-name",
    event = "VeryLazy",  -- or cmd, ft, keys for lazy loading
    opts = {},
  },
}
```

Plugins are auto-discovered from `lua/plugins/` via `{ import = "plugins" }` in lazy.lua.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnnatarHe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
