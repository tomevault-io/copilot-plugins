## my-nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview
This is a modular Neovim configuration using Lua, built with lazy.nvim as the plugin manager. The configuration follows modern Neovim best practices with separate modules for different concerns.

## Architecture
- **Entry Point**: `init.lua` - Bootstraps lazy.nvim and loads core configuration modules
- **Core Configuration**: Located in `lua/core/` with separate files for:
  - `options.lua`: Editor settings, appearance, file handling
  - `keymaps.lua`: Global keybindings and leader key setup
  - `autocmds.lua`: Automatic commands for various behaviors
- **Plugins**: Managed by lazy.nvim, defined in `init.lua` with individual configuration files per plugin in `lua/plugins/`
- **LSP**: Configured in `lua/lsp/` with separate files for LSP setup and completion

## Key Commands & Usage
- **Plugin Management**: 
  - Install plugins: `:Lazy install`
  - Update plugins: `:Lazy update`
  - Clean unused plugins: `:Lazy clean`
  - Plugin status: `:Lazy`
- **LSP Management**:
  - Install LSP servers: `:Mason`
  - LSP info: `:LspInfo`
- **Common Operations**:
  - File explorer: `<leader>e` (nvim-tree)
  - Fuzzy find files: `<leader>ff` (telescope)
  - Live grep: `<leader>fg` (telescope)
  - Toggle terminal: `<leader>g` (toggleterm)

## Configuration Structure
```
~/.config/nvim/
├── init.lua                 # Main entry point with lazy.nvim
├── lua/
│   ├── core/
│   │   ├── options.lua      # Core editor settings
│   │   ├── keymaps.lua      # Global keybindings
│   │   └── autocmds.lua     # Auto commands
│   ├── plugins/             # Plugin configurations
│   │   ├── *.lua            # Individual plugin configs
│   └── lsp/                 # LSP configuration
│       ├── init.lua         # LSP server setup
│       └── cmp.lua          # Completion setup
```

## Important Paths
- **Plugin data**: `~/.local/share/nvim/lazy/`
- **Mason binaries**: `~/.local/share/nvim/mason/bin/`
- **Configuration**: `~/.config/nvim/`

## Development Notes
- All plugins use lazy.nvim for management
- Keymaps use `<leader>` (space) as prefix
- LSP servers are installed via Mason for consistency
- Treesitter provides syntax highlighting for 18+ languages
- Git integration includes gitsigns and terminal integration

## Claude Guidance
- The users are The Minimalists! So never automatically add unnecessary stuff to their nvim/vim

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GowayLee)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GowayLee)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
