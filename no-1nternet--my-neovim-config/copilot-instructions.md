## my-neovim-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a Neovim configuration that uses Lazy.nvim as the plugin manager. The configuration follows a modular structure:

- `init.lua` - Main configuration entry point, sets up Lazy.nvim and defines plugin specifications
- `lua/lsp.lua` - LSP configuration for language servers (currently lua-language-server)
- `lua/plugin/completion.lua` - Completion setup using blink.cmp
- `after/plugin/` - Plugin-specific configurations that load after plugins are installed
  - `telescope.lua` - File finder and search keymaps
  - `treesitter.lua` - Syntax highlighting and parsing configuration
  - `undotree.lua` - Undo tree visualization keymaps

## Plugin Management

Uses Lazy.nvim for plugin management. Key plugins include:
- **tokyonight.nvim** - Colorscheme (loads on startup)
- **telescope.nvim** - Fuzzy finder for files and content
- **nvim-treesitter** - Syntax highlighting and code parsing
- **blink.cmp** - Completion engine with LSP integration
- **undotree** - Undo history visualization
- **treesj** - Join/split code blocks
- **neorg** - Note-taking (lazy-loaded on .norg files)

## Key Mappings

- `<Space>` - Leader key
- `<leader>vv` - Open file explorer (vim.cmd.Ex)
- `<leader>ff` - Telescope find files
- `<C-p>` - Telescope git files
- `<leader>ps` - Telescope grep search
- `<leader>u` - Toggle undotree
- `J` - TreeSJ toggle (join/split)

## LSP Configuration

LSP is configured in `lua/lsp.lua` with:
- Auto-completion enabled on attach
- Auto-formatting on save (when supported)
- Currently configured for Lua language server only

## Development Commands

Since this is a Neovim configuration, development involves:
- Editing Lua configuration files
- Testing changes by restarting Neovim or sourcing files
- Managing plugins through Lazy.nvim interface (`:Lazy`)
- Updating treesitter parsers with `:TSUpdate`

The configuration automatically bootstraps Lazy.nvim if not present and enables automatic plugin update checking.

---
> Source: [no-1nternet/my-neovim-config](https://github.com/no-1nternet/my-neovim-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
