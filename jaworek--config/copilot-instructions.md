## config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal macOS system configuration built with **Nix Flakes** and **nix-darwin**. It declaratively manages system settings, applications, and dotfiles using the Nix package manager and Home Manager. The configuration supports two profiles: `john` (personal) and `work`.

## Common Commands

### System Rebuild Commands
- `sudo /Users/john/.config/nix/rebuild-and-switch.sh` - Rebuild and switch to personal configuration
- `sudo /Users/john/.config/nix/rebuild-and-switch-work.sh` - Rebuild and switch to work configuration
- `darwin-rebuild-switch` - Alias for personal config rebuild
- `darwin-rebuild-switch-work` - Alias for work config rebuild

### Maintenance Commands
- `nix flake update` - Update flake.lock with latest package versions
- `flake-update` - Alias for flake update
- `sudo nix-collect-garbage --delete-older-than 7d` - Clean up old Nix generations
- `darwin-cleanup` - Alias for cleanup command

### Development Aliases (defined in configuration.nix)
- `g` → `git`
- `gs` → `git status`  
- `gc` → `git commit`
- `gaa` → `git add --all`
- `gpu` → `git push`
- `gpuf` → `git push --force-with-lease`

## Architecture

### Core Configuration Files
- **`flake.nix`** - Main flake definition with system outputs for `john` and `work` configurations
- **`configuration.nix`** - System-wide Darwin settings (fonts, security, shell aliases, environment)
- **`home.nix`** - Home Manager configuration for user-level settings and dotfile symlinks

### Modular Structure
```
apps/
├── common.nix     # Shared development tools (neovim, git, tmux, etc.)
├── personal.nix   # Personal applications (Discord, Blender, Obsidian)
└── work.nix       # Work-specific tools (1Password, EAS CLI)

macos/
├── homebrew.nix   # Homebrew packages and casks
├── dock.nix       # macOS dock configuration  
└── karabiner.nix  # Keyboard remapping

home-manager/
├── git.nix        # Git config with work/personal email switching
├── ghostty.nix    # Terminal emulator settings
└── zsh.nix        # Shell configuration
```

### Key Behaviors
- Two-profile system (`john`/`work`) with different application sets
- Automatic dotfile management via Home Manager symlinks
- Integrated tmux configuration with plugin management (TPM)
- macOS-specific settings (Touch ID sudo, dock, trackpad)
- Development environment with Go, Rust, Node.js, Java support

## Development Environment

### Installed Languages & Tools
- **Languages**: Go, Rust, Node.js, Java (Zulu 17), Bun
- **Editors**: Neovim, Zed, WebStorm, Cursor, Claude Code
- **CLI Tools**: ripgrep, fd, fzf, lazygit, gh, tmux with plugins
- **Mobile Dev**: React Native tools, Android SDK paths, Watchman
- **Containers**: Podman (aliased as `docker`)

### Configuration Management
- **Dotfiles** managed in `/dotfiles/` directory with symlinks via Home Manager
- **tmux** configured with TPM plugin manager, sensible defaults, navigation helpers
- **Shell** enhanced with Starship prompt and extensive aliases
- **Git** configured with delta diff viewer and context-aware email switching

## Making Changes

When modifying this configuration:
1. Edit the relevant `.nix` files
2. Test changes with `darwin-rebuild-switch` (personal) or `darwin-rebuild-switch-work`
3. Update flake inputs periodically with `nix flake update`
4. Clean up old generations regularly with `darwin-cleanup`

The system automatically manages dotfile symlinks, so changes to files in `/dotfiles/` require a rebuild to take effect.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jaworek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
