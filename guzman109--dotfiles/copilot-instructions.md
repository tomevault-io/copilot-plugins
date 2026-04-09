## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a macOS dotfiles repository managing configuration for development tools and terminal environments. The primary structure uses a `config/.config/` directory containing all application configurations.

## Repository Structure

- `config/.config/` - Main configuration directory containing:
  - `fish/` - Fish shell configuration and completions
  - `nvim/` - Neovim (LazyVim) configuration
  - `ghostty/` - Ghostty terminal emulator config
  - `kitty/` - Kitty terminal emulator config
  - `starship/` - Starship prompt configuration

## Key Configuration Files

### Fish Shell (`config/.config/fish/`)
- `config.fish` - Main shell configuration with:
  - Homebrew environment setup (macOS)
  - PATH configuration for Cargo, Bun, and Bob (nvim version manager)
  - Starship prompt initialization
  - Shell aliases (eza for ls, bat for cat)
- `completions/` - Auto-completion scripts for various tools (uv, bun, swift, colima)
- `conf.d/` - Additional configuration snippets loaded on shell startup

### Neovim (`config/.config/nvim/`)
Based on LazyVim distribution. Configuration uses Lua plugin system in `lua/plugins/`:

- `lsp.lua` - LSP configurations with detailed setup for:
  - Swift (sourcekit LSP with custom root detection)
  - Python (commented ty + ruff setup)
  - Zig (commented zls setup)
  - Custom diagnostic formatting showing source prefixes
  - Language-specific priority and file settings

- `coding.lua` - Coding tools:
  - Claude Code integration (`coder/claudecode.nvim`) with extensive keybindings under `<leader>a`
  - UFO folding with LSP and indent providers

- `format.lua` - Conform.nvim formatters:
  - Swift: swiftformat (Homebrew installed at `/opt/homebrew/bin/swiftformat`)
  - Python: ruff_format (installed at `/Users/guzman.109/.local/bin/ruff`)
  - Zig: zigfmt
  - Format on save enabled with 500ms timeout

- `lint.lua` - Linting configuration

- `theme.lua` - Theme management:
  - Catppuccin and Tokyo Night themes
  - Transparent background enabled for Catppuccin
  - Themery plugin for theme switching (`<leader>tt`)
  - Rainbow delimiters plugin

- `snacks.lua` - Snacks.nvim dashboard with Git integration:
  - Custom dashboard sections showing GitHub notifications, issues, PRs
  - Git status terminal section
  - Uses `gh` CLI and `colorscript`

- `lua/config/options.lua` - Global settings:
  - Auto-format disabled by default (`vim.g.autoformat = false`)
  - Root detection set to current working directory
  - Python LSP set to basedpyright and ruff

### Terminal Emulators

**Ghostty** (`config/.config/ghostty/config`):
- MonoLisa font at 14pt
- Catppuccin theme (light/dark auto-switching)
- Background opacity 0.8 with blur
- Fish shell integration
- Fullscreen by default

**Kitty** (`config/.config/kitty/`):
- `kitty.conf` - Main configuration
- `dark-theme.auto.conf` / `light-theme.auto.conf` - Theme files

### Starship Prompt (`config/.config/starship/starship.toml`)
- Custom format with time, username, directory, git status
- Language version indicators for many programming languages
- Custom success/error symbols
- 3-second command timeout
- 12-hour time format

## Development Workflow

This is a dotfiles repository - there are no build, test, or lint commands at the repository level. Changes are typically:

1. Edit configuration files directly
2. Test by reloading the affected application (e.g., `source ~/.config/fish/config.fish`, reopen nvim)
3. Commit changes with git

## Important Notes

- **LazyVim**: Neovim uses LazyVim distribution. Plugin modifications should follow LazyVim patterns (returning plugin specs from `lua/plugins/*.lua`)
- **LSP Priority**: Swift, Python, and Zig have custom LSP priority setup to ensure primary language servers attach first
- **Paths**: Several configurations use absolute paths (e.g., formatter paths, local bins). When modifying, maintain these absolute references
- **macOS-specific**: Fish config checks for Darwin (macOS) before setting up Homebrew
- **Shell Integration**: Both Ghostty and Kitty are configured for Fish shell integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guzman109)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/guzman109)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
