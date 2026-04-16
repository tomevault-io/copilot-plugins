## nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a Neovim configuration that supports both standalone Neovim and VS Code/Cursor environments:

- **Dual Environment Support**: `init.lua` detects VS Code via `vim.g.vscode` and loads either `native-config.lua` or `vscode-config.lua`
- **Plugin Management**: Uses Lazy.nvim for plugin management with specs imported from `lua/plugins/`
- **Configuration Structure**: 
  - `lua/vim-options.lua`: Core vim settings and keymaps
  - `lua/plugins/`: Individual plugin configurations
  - `lua/config/lsp-setup.lua`: LSP configuration logic

## Common Commands

### Initial Setup
- Install Neovim: `brew install neovim`
- Install dependencies: `brew install ripgrep fd` (for Telescope)
- Clone to `~/.config/nvim`

### Language Server Setup
- Run `:Mason` in Neovim
- Install packages listed in `Mason.txt`:
  - typescript-language-server
  - eslint_d, prettier, shfmt, stylua
  - lua-language-server, json-lsp, eslint-lsp
  - vue-language-server (if needed)

### Copilot Setup
- Run `:Copilot setup` and follow authentication steps
- Requires GitHub account with Copilot enabled

### Troubleshooting
- Use `:checkhealth [PACKAGE_NAME]` to diagnose issues
- VS Code integration automatically applies when detected

## Key Configuration Files

- `lua/plugins/lsp-config.lua`: LSP setup with Mason integration
- `lua/plugins/telescope.lua`: File/code search configuration  
- `lua/plugins/copilot.lua` & `lua/plugins/copilotChat.lua`: AI assistance
- `lua/vscode-config.lua`: VS Code-specific keymaps and actions

## Development Notes

- Leader key: `<space>`
- Custom keymaps include window navigation (`<leader>hjkl`) and reload current file (`<leader>n`)
- VS Code mode uses VS Code actions for navigation and commands
- Plugin installation handled automatically by Lazy.nvim
- LSP configuration is project-aware via `lsp-setup.lua`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kei95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
