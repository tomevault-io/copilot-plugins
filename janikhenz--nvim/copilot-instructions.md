## nvim

> This repository contains a personal Neovim configuration, designed to provide an efficient and customized development environment. The configuration is written in Lua and leverages Neovim's built-in package manager (`vim.pack.add`) for plugin management. It includes setups for Language Server Protocol (LSP) using `mason.nvim` and `mason-lspconfig.nvim`, fuzzy finding with Telescope and `fzf-native`, file navigation with Oil.nvim, and various other productivity-enhancing plugins like Harpoon, Treesitter, and Markdown Preview. The configuration also includes specific keybindings for efficient interaction with the Gemini CLI.

# Neovim Configuration

## Project Overview
This repository contains a personal Neovim configuration, designed to provide an efficient and customized development environment. The configuration is written in Lua and leverages Neovim's built-in package manager (`vim.pack.add`) for plugin management. It includes setups for Language Server Protocol (LSP) using `mason.nvim` and `mason-lspconfig.nvim`, fuzzy finding with Telescope and `fzf-native`, file navigation with Oil.nvim, and various other productivity-enhancing plugins like Harpoon, Treesitter, and Markdown Preview. The configuration also includes specific keybindings for efficient interaction with the Gemini CLI.

## Key Features
*   **Plugin Management:** Uses `vim.pack.add` for declarative plugin installation.
*   **Language Server Protocol (LSP):** Configured with `mason.nvim` and `mason-lspconfig` for robust code intelligence, including specific setups for `lua_ls` and `clangd`. Keybindings for LSP functionalities like hover, go to definition, rename, code action, and format are provided.
*   **Fuzzy Finding:** Integrates Telescope with `fzf-native` for fast file, buffer, and grep searching. Includes a check to build `fzf-native` if its shared library is not found.
*   **File Navigation:** Utilizes `oil.nvim` for enhanced file system browsing.
*   **Code Completion:** Includes `blink.cmp` for autocomplete functionality.
*   **Syntax Highlighting:** Configured with `nvim-treesitter`.
*   **Key Remappings:** Custom keybindings for efficient navigation, search, editing, and LSP interactions. Specific mappings are included for interacting with the Gemini CLI, such as `<leader>gg` to open a terminal with `gemini`.
*   **Theming:** Uses `catppuccin/nvim` for a pleasant visual experience, with the `macchiato` colorscheme.
*   **General Neovim Settings:** Configures indentation, line numbers, leader keys, clipboard, and UI appearance.

## Structure
The configuration follows a modular structure:
*   `init.lua`: The main entry point for Neovim, responsible for loading plugins and sourcing other configuration files.
*   `lua/options.lua`: Contains general Neovim settings and options.
*   `lua/remaps.lua`: Defines custom key mappings.
*   `lua/gemini.lua`: Contains specific configurations for interacting with the Gemini CLI.
*   `lua/plugins/`: A directory containing individual Lua files for configuring specific plugins (e.g., `lsp.lua`, `telescope.lua`, `oil.lua`, `harpoon.lua`, `treesitter.lua`, `markdown.lua`, `autocomplete.lua`).

## Installation and Usage

### Requirements
*   Neovim (v0.8.0 or later, preferably latest stable)
*   `git`

### Installation
1.  **Backup (Optional):** If you have an existing Neovim configuration, back it up:
    ```bash
    mv ~/.config/nvim ~/.config/nvim_backup
    ```
2.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/your-nvim-config.git ~/.config/nvim
    ```
    (Note: Replace `https://github.com/your-username/your-nvim-config.git` with the actual URL of this repository if it were hosted.)
3.  **Open Neovim:**
    ```bash
    nvim
    ```
    Upon first launch, Neovim will automatically install the plugins defined in `init.lua` via `vim.pack.add`. Mason will also install LSP servers as configured.

### Building Plugins (if necessary)
Some plugins, like `telescope-fzf-native.nvim`, require compilation. The `lua/plugins/telescope.lua` configuration includes a check to build `fzf-native` if its shared library is not found.

### Updating Plugins
To update all plugins, you can run:
```lua
:lua vim.cmd('packadd all')
:lua vim.cmd('helptags ALL')
```
(Note: `vim.pack.add` handles updates by pulling the latest from the configured source.)

## Development Conventions
*   **Lua-based Configuration:** All configuration is written in Lua.
*   **Modularity:** Configuration is split into logical files within the `lua/` directory for better organization.
*   **Conditional Loading:** Some UI-specific or potentially conflicting plugins/settings are conditionally loaded (e.g., based on whether Neovim is running inside VS Code, though this was not explicitly observed in the files read, it was mentioned in the initial context).
*   **Keybinding Documentation:** Key remaps are typically documented with comments or descriptive messages where appropriate in `lua/remaps.lua` and plugin-specific configuration files.
*   **LSP Configuration:** LSP servers are configured via `mason.nvim` and `mason-lspconfig.nvim`, with specific settings for `lua_ls` and `clangd` to enhance code intelligence.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JanikHenz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
