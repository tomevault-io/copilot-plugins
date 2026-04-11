## dotfiles

> This is a personal dotfiles repository for managing development environment configurations across macOS, WSL2, and Windows.

# Gemini Workspace Overview

This is a personal dotfiles repository for managing development environment configurations across macOS, WSL2, and Windows.

## Project Structure

The repository is organized by platform, with shared configurations in the `common` directory.

- **`common/`**: Shared configurations for tools like `git`, `tmux`, `starship`, `zellij`, `pet`, `nvim`, `vim`, and `python`.
- **`platforms/macOS/`**: macOS-specific configurations, including shell profiles (`.bash_profile`, `.zshrc`), a `Brewfile` for package management, and system defaults.
- **`common/nvim/`**: Neovim configuration using Lua and the `lazy.nvim` plugin manager (cross-platform).
- **`common/python/`**: Python linting and code style configurations (`flake8`, `pycodestyle`, `pylintrc`).
- **`common/vim/`**: Legacy Vim configuration using `dein.vim` (cross-platform).
- **`platforms/windows/`**: Windows-specific configurations, such as AutoHotkey scripts.
- **`platforms/WSL2/`**: Configurations for Windows Subsystem for Linux.

## Initial Setup

The main setup script, `dotfilesLink.sh`, orchestrates the linking of configuration files.

```bash
# Run the main setup script
sh ~/dotfiles/dotfilesLink.sh
```

### Platform-Specific Setup

You can also run setup scripts for specific platforms or tools:

- **macOS**: `sh ~/dotfiles/platforms/macOS/setup_macos.sh`
- **Neovim**: `sh ~/dotfiles/common/nvim/setup_nvim.sh`
- **Shared Tools**: `sh ~/dotfiles/common/synbolic_link.sh` (includes nvim, vim)
- **Python**: `sh ~/dotfiles/common/python/symbolic_link.sh`
- **WSL2**: `sh ~/dotfiles/platforms/WSL2/setup_wsl2.sh`

### macOS Package Installation

The `platforms/macOS/Brewfile` lists packages to be installed with Homebrew.

```bash
cd ~/dotfiles/platforms/macOS
brew bundle
```

## Neovim Configuration

The Neovim configuration is located in the `common/nvim/` directory and uses `lazy.nvim` for plugin management.

- **`init.lua`**: The entry point for the Neovim configuration.
- **`lua/plugins.lua`**: The central file for plugin management.
- **`lua/pluginconfig/`**: Directory containing individual plugin configurations.

### Neovim Plugin Management

- Plugins are automatically installed on the first launch.
- To manually update plugins, run `:Lazy sync` in Neovim.
- To update Treesitter parsers, run `:TSUpdate`.
- To manage LSP servers, use `:Mason`.

## Development Conventions

- Shell scripts are written in POSIX `#!/bin/sh` and use 2-space indentation.
- Lua modules also use 2-space indentation.
- Python code follows the rules defined in the `common/python/` directory.
- Commit messages should be short and imperative (e.g., "add feature for X").

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/N0nki)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/N0nki)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
