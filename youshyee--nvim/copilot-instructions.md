## nvim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a comprehensive Neovim configuration for a development environment. The configuration uses vim-plug for plugin management and includes extensive language support through CoC (Conquer of Completion) and various LSP servers.

## Configuration Structure

- **init.vim**: Main configuration file that sources other modules and sets up the environment
- **plugins.vim**: Plugin definitions using vim-plug
- **general/settings.vim**: Core editor settings and behaviors
- **key.vim**: Key mappings and custom commands
- **functions.vim**: Custom utility functions
- **plugconfig/**: Individual plugin configuration files
- **coc-settings.json**: CoC (Conquer of Completion) configuration with LSP settings
- **UltiSnips/**: Custom snippet definitions
- **_machine_specific.vim**: Machine-specific settings (copied from default on first run)

## Key Features

### Language Support
- **Python**: PyRight LSP, debugpy support via vimspector
- **JavaScript/TypeScript**: CoC with prettier formatting
- **Go**: gopls LSP integration
- **LaTeX**: texlab LSP support
- **Lua**: lua-lsp integration
- **C/C++**: ccls integration
- **Shell**: bash-language-server
- **Web**: HTML, CSS, JSON formatting and completion

### Key Plugins
- **vim-plug**: Plugin manager
- **coc.nvim**: LSP client with multiple language servers
- **fzf/fzf.vim**: Fuzzy finding
- **vim-airline**: Status line
- **vim-which-key**: Key binding documentation
- **ultisnips**: Snippet engine
- **vimtex**: LaTeX support
- **gpt.nvim**: AI assistant integration
- **copilot.vim**: GitHub Copilot integration

### Development Workflow
- **Git integration**: fugitive, gitgutter, vim-signify
- **File management**: ranger integration, vim-rooter
- **Code formatting**: autoformat, prettier
- **Debugging**: vimspector with Python support
- **Terminal**: floaterm for floating terminal windows
- **Search**: ripgrep integration

## Prerequisites

Before using this configuration, install:
- `pynvim` (pip)
- `ripgrep` (system package manager)
- `ranger` (system package manager)
- `fzf` (system package manager)
- `nodejs` and `npm install -g neovim`
- `figlet` (for ASCII art)
- `xclip` (Linux clipboard support)
- Nerd fonts (optional but recommended)
- `debugpy` (for Python debugging)

## Configuration Files

### Machine-Specific Settings
- Edit `_machine_specific.vim` for Python interpreter paths and other machine-specific settings
- This file is automatically created from `default_configs/_machine_specific_default.vim` on first run

### Plugin Configuration
- Individual plugin configurations are in `plugconfig/` directory
- Each plugin has its own configuration file (e.g., `coc.vim`, `fzf.vim`)

### Key Mappings
- Custom key mappings are defined in `key.vim`
- Plugin-specific mappings are in their respective `plugconfig/` files

## Common Commands

### Plugin Management
- `:PlugInstall` - Install plugins
- `:PlugUpdate` - Update plugins
- `:PlugClean` - Remove unused plugins

### CoC Commands
- `:CocInstall coc-python` - Install Python language server
- `:CocConfig` - Open CoC configuration
- `:CocInfo` - Show CoC information

### Git Integration
- `:G` - Open fugitive git status
- `:Gblame` - Show git blame
- `:Gdiff` - Show git diff

### File Operations
- `:Ranger` - Open ranger file manager
- `:Files` - Fuzzy find files
- `:Rg` - Ripgrep search

## Development Notes

- The configuration uses true color support with `termguicolors`
- Current color scheme is `kanagawa` (changeable in init.vim)
- Auto-completion is handled by CoC with custom suggestions
- Snippets are managed by UltiSnips with custom definitions in `UltiSnips/`
- The configuration includes extensive markdown support with spell checking
- Copilot integration requires `:Copilot setup` after installation

## Testing Configuration

After making changes:
1. Run `:checkhealth` to verify Neovim health
2. Test plugin functionality with `:PlugStatus`
3. Verify LSP servers are working with `:CocInfo`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/youshyee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
