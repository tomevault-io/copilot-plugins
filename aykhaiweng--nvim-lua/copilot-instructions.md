## nvim-lua

> A highly modular and organized Neovim configuration written in Lua, leveraging `lazy.nvim` for plugin management and `mason.nvim` for external tool integration.

# Neovim Configuration Overview

A highly modular and organized Neovim configuration written in Lua, leveraging `lazy.nvim` for plugin management and `mason.nvim` for external tool integration.

## Architecture

- **`init.lua`**: Entry point. Sets leaders and loads core modules followed by plugin initialization.
- **`lua/core/`**: Fundamental Neovim settings.
    - `options.lua`: Editor options (indentation, search, UI, etc.).
    - `keymaps.lua`: Global keybindings.
    - `autocmds.lua`: Custom autocommands.
    - `colors.lua`: Colorscheme and highlight overrides.
    - `terminals.lua`: Terminal-specific configuration.
    - `utils.lua`: Shared utility functions (e.g., project-level file ignores).
- **`lua/lazy_init.lua`**: Bootstraps `lazy.nvim` and orchestrates plugin imports from categorized subdirectories.
- **`lua/plugins/`**: Categorized plugin configurations (e.g., `ai/`, `editor/`, `git/`, `lsp/`, `navigation/`, `ui/`).
- **`lua/lsp/`**: Specific LSP server configurations (e.g., `basedpyright.lua`).
- **`after/ftplugin/`**: Filetype-specific settings (e.g., `python.lua`, `lua.lua`).
- **`snippets/`**: Custom JSON snippets.

## Technologies

- **Plugin Manager**: [lazy.nvim](https://github.com/folke/lazy.nvim)
- **LSP Support**: `nvim-lspconfig`, `mason.nvim`, `mason-lspconfig.nvim`.
- **Syntax Highlighting**: `nvim-treesitter`.
- **UI & Navigation**: `telescope.nvim`, `neo-tree.nvim`, `lualine.nvim`, `snacks.nvim`.
- **AI Integration**: `codecompanion.nvim`, `gemini-companion.lua`.
- **Formatting & Linting**: `conform.nvim`, `nvim-lint`.

## Core Conventions

### Keybindings
- **Leader Key**: ` ` (Space)
- **Local Leader**: `\` (Backslash)
- **General**:
    - `C-c` and `C-q` map to `<Esc>` in Insert and Visual modes.
    - `Y` and `P` are optimized for system clipboard interaction and non-overwriting pastes.
    - `<leader>th/j/k/l`: Tab navigation.
    - `<M-h/j/k/l>`: Window resizing and cursor movement in insert mode.
- **LSP** (Buffer-local):
    - `K`: Hover.
    - `gd`: Go to definition.
    - `gr`: Go to references.
    - `gl`: Show line diagnostics.
    - `<leader>rn`: Rename symbol.

### Development Environment
- **Python**: Expects a Neovim-specific python environment at `~/.pyenv/versions/neovim3/bin/python3`.
- **Project Configuration**: Supports project-level `.nvim.lua` for local overrides and file ignores (via `lua/core/utils.lua`).
- **Appearance**: Default colorscheme is `catppuccin`.

## Common Commands

- `:Lazy`: Open the plugin manager UI.
- `:Mason`: Manage LSP servers, linters, and formatters.
- `:LspInfo`: Check status of active language servers.
- `<leader>ff`: Auto-format current buffer using `gg=G`.
- `<leader>ew`: Search and replace word under cursor.

## Adding New Plugins

1.  Identify the appropriate category in `lua/plugins/`.
2.  Create or update a Lua file in that category.
3.  Ensure the file returns a table containing the `lazy.nvim` plugin specification.
4.  If the category is new, add `{ import = "plugins.category_name" }` to `lua/lazy_init.lua`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aykhaiweng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
