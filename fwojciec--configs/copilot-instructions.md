## configs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains personal configuration files (dotfiles) for various development tools and terminal environments on macOS. It includes configurations for:

- Terminal emulators (Kitty, Alacritty, Ghostty, iTerm2)
- Neovim editor with extensive customization
- Keyboard remapping with Karabiner
- Various utility scripts for system management
- Shell prompt customization with Starship

## Key Components

### Terminal Configuration

- Multiple terminal emulators with similar configurations
- Preference for Iosevka Term font and dark themes (particularly Gruvbox Material)
- Consistent styling and behavior across terminals

### Neovim Setup

The Neovim configuration uses a modern Lua-based approach with these key components:

- Package management with lazy.nvim
- LSP support for multiple languages (Go, TypeScript, Python, Lua, etc.)
- Code completion with nvim-cmp
- Fuzzy finding with Telescope
- Syntax highlighting with Treesitter
- Code formatting with formatter.nvim
- Testing with vim-test
- AI assistance with codecompanion.nvim and GitHub Copilot

Important key mappings:
- Leader key is space
- `<leader>w` - save file
- `<leader>?` - recent files
- `<leader><space>` - find buffers
- `<leader>sg` - live grep
- `<leader>t`/`<leader>T` - run tests
- LSP functions mapped to intuitive keys (gd, K, etc.)

### System Utilities

The repository contains scripts for:
- Keyboard customization (swapping Command/Option keys)
- macOS optimizations (reducing dock animation delay)
- Photography workflow helpers
- Development environment setup

## Working with this Repository

When making changes to these configurations:

1. Test changes in isolation before committing
2. Maintain the existing modular structure in Neovim config
3. Preserve consistent styling across different tools
4. Keep language-specific customizations separate

For Neovim changes, understand the module structure in `lua/fw/` directory and language-specific settings in `after/ftplugin/`.

Utility scripts are standalone and should be documented with comments explaining their purpose and usage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fwojciec) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
