## dotfiles

> This file provides guidance to Gemini when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Project Overview

This is a dotfiles management system for Linux environments using Nix and Home Manager for declarative configuration. It supports multiple host types: desktops, servers, and WSL instances.

## Common Commands

### Fresh Install (Primary Method)
```bash
./apply.sh --nix-host <host-name>
# Example: ./apply.sh --nix-host personal-desktop
```
Available hosts are defined in `nix/hosts.json`: personal-desktop, work-desktop, personal-wsl, work-wsl, personal-server, work-server.

### Apply Nix Configuration Changes
```bash
home-manager switch --flake ~/dot/nix#<host-name>
```

### Python CLI Tool
```bash
dot provision [TAGS]    # Run provisioners
dot link                # Create symlinks (legacy)
dot clean               # Remove symlinks
dot tidy                # Format Python code (black, isort, autoflake)
```

### Type Checking and Linting
```bash
make mypy               # Run mypy on all Python files
```

### Theming
```bash
set-theme <scheme-name>  # Apply base16 color scheme (e.g., outrun-dark)
flavours update all      # Required first time setup
```

## Architecture

### Nix Configuration (`nix/`)
- `flake.nix` - Entry point; generates Home Manager configurations from `hosts.json`
- `hosts.json` - Defines hosts and their roles (core, desktop, gaming, wsl)
- `home/roles/` - Host-type configurations (Ansible-style naming):
  - `core.nix` - Base packages (neovim, git, fzf, tmux, Python tools)
  - `desktop.nix` - GUI packages (i3, kitty, rofi, mpd, media tools)
  - `gaming.nix` - Gaming packages
  - `wsl.nix` - WSL-specific configuration
- `home/lib/` - Shared modules imported by roles:
  - `dotfiles-links.nix` - Maps config files from `config/` to home directory
  - `python-environment.nix` - Builds unified Python environment
- `home/features/` - Specific feature modules (e.g., docker) that can be shared across roles

### Configuration Files (`config/`)
Application configs that get linked to home directory via Home Manager:
- `bash/` - Shell configuration
- `nvim/` - Neovim configuration
- `i3`, `sway` - Window manager configs
- `kitty.conf`, `alacritty/`, `wezterm.lua` - Terminal emulators
- `flavours/` - Base16 theming (schemes in `schemes/custom/`, templates in `templates/custom/`)

### Python CLI (`cli/`)
- `dot.py` - Main entry point with argparse
- `commands/` - Command implementations (provision, link, tidy, lint, etc.)
- `lib/provision/` - Provisioners for apt, pip, nodejs, docker, neovim, etc.
- `lib/common/` - Shared utilities (shell, git, logging, file operations)

### Utility Scripts (`bin/`)
Standalone scripts: `set-theme`, `i3-util.sh`, `startup.sh`, `fuzzy-fm`, etc.

## Key Files

- `apply.sh` - Bootstrap script for fresh installs (installs Nix, applies Home Manager)
- `links.json` - Legacy dotfile symlink mappings (now mostly handled by `dotfiles-links.nix`)
- `nix/hosts.json` - Host definitions with roles; drives the entire Nix configuration

## Code Style

Python code uses black, isort, and autoflake. Run `dot tidy` before committing Python changes. Type hints are encouraged; validate with `make mypy`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pcewing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pcewing)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
