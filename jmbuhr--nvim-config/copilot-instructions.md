## nvim-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal Neovim configuration focused on data science and literate programming, particularly for R, Python, Quarto, and Jupyter workflows. The config is built around the Lazy.nvim plugin manager and emphasizes code execution in notebooks and terminals.

## Core Architecture

### Configuration Structure
- **init.lua**: Main entry point that loads core modules and sets colorscheme
- **lua/config/**: Core configuration modules
  - `global.lua`: Global options, statusline, filetype mappings
  - `lazy.lua`: Lazy.nvim bootstrap and plugin loading configuration
  - `keymap.lua`: Comprehensive keybinding definitions with which-key integration
  - `autocommands.lua`: Vim autocommands
  - `redir.lua`: Output redirection utilities

### Plugin Organization
Plugins are organized by functionality in `lua/plugins/`:
- `ai.lua`: AI/copilot integrations
- `lsp.lua`: Language server protocol configuration (Mason, nvim-lspconfig)
- `completion.lua`: Autocompletion setup
- `quarto.lua`: Quarto document support
- `treesitter.lua`: Syntax highlighting and parsing
- `ui.lua`: UI enhancements, statusline, file explorers
- `editing.lua`: Text editing enhancements
- `git.lua`: Git integration
- `debugging.lua`: Debug adapter protocol
- `notes.lua`: Note-taking and knowledge management
- `common.lua`: Shared utilities and dependencies

### Key Technologies
- **Otter.nvim**: Provides LSP features for embedded code in notebooks/Quarto documents
- **Quarto.nvim**: First-party Quarto document support
- **vim-slime**: Code execution in terminal/REPL integration
- **Molten**: Jupyter kernel integration for direct notebook execution
- **Mason**: LSP/tool installation management

## Development Commands

### Code Formatting
```bash
# Format Lua code with stylua (config in stylua.toml)
stylua .
```

### Plugin Management
```bash
# Within Neovim - access Lazy.nvim package manager
:Lazy
```

## Key Features & Workflows

### Code Execution
- **Cell execution**: `<C-cr>` or `<S-cr>` sends current cell to terminal/REPL
- **Language switching**: Automatic R/Python REPL switching via reticulate in R mode
- **Terminal integration**: Creates language-specific terminals (`<leader>cp`, `<leader>cr`, etc.)

### Document Support
- **Quarto**: Full integration for rendering and preview
- **Multi-language**: R, Python, Julia, Bash, Observable JS code chunks
- **Otter LSP**: Language server features within embedded code blocks

### Code Chunk Management
- Insert code chunks: `<leader>op` (Python), `<leader>or` (R), etc.
- Insert plain chunks: `<leader>Op`, `<leader>Or` (without curly braces)
- Alt shortcuts: `<m-i>` (R chunk), `<m-I>` (Python chunk)

### Navigation & Search
- Telescope integration for fuzzy finding (`<leader>f*`)
- Git integration with worktree support
- LSP-powered symbol navigation

## Configuration Conventions

### Keybinding Patterns
- `<leader>` (space) prefix for most commands
- Grouped by functionality: `<leader>c` (code), `<leader>g` (git), `<leader>f` (find)
- Terminal control sequences mapped for kitty/wezterm compatibility
- which-key integration provides discoverable keybindings

### Code Style
- 2-space indentation for Lua files
- StyLua formatting with specific configuration (stylua.toml)
- Conceal levels: DefaultConcealLevel (0) and FullConcealLevel (3)

### Plugin Development
- Development plugins loaded from `~/projects` directory
- Local plugins for: quarto-nvim, otter.nvim, cmp-pandoc-references, telescope-zotero.nvim

## Special Integrations

### Terminal Requirements
- Requires kitty or wezterm for image display (image.nvim)
- tmux >= 3.3a for image support in tmux sessions
- Terminal font: Nerd Font for icons

### R/Python Workflow
- Automatic reticulate switching when moving between R/Python chunks
- R dataframe viewing with DT::datatable integration
- Context-aware code execution based on cursor position in documents

## Plugin Documentation Reference

When working with plugins, documentation can always be referenced from their GitHub README files using the pattern `<username>/<repo>` (e.g., `folke/lazy.nvim`, `quarto-dev/quarto-nvim`).

---
> Source: [jmbuhr/nvim-config](https://github.com/jmbuhr/nvim-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
