## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Overview

This is a personal dotfiles repository for an Arch Linux system with a Wayland-based desktop environment. The configuration uses Sway (i3-compatible Wayland compositor) as the window manager with Catppuccin Mocha color scheme throughout.

## Architecture

### Window Manager & Desktop Environment
- **Sway**: Main window manager configuration at `.config/sway/config`
  - Mod key: Super/Windows key (Mod4)
  - Terminal: Alacritty
  - Application launcher: Wofi
  - Catppuccin Mocha color scheme with #89b4fa (blue) for focused windows
  - 8px inner gaps, 4px outer gaps, 2px borders

### Status Bar & UI
- **Waybar**: Top bar configuration at `.config/waybar/config` and `.config/waybar/style.css`
  - Modules: workspaces, window title, clock, bluetooth, network, pulseaudio, CPU, memory, tray
  - Uses Nerd Font icons for visual elements
  - Note: Waybar v0.9.x does not support advanced bluetooth/temperature format strings; config is simplified for compatibility

- **Wofi**: Application launcher at `.config/wofi/config`
  - 600x400 centered window with dark theme

### Terminal & Shell
- **Alacritty**: Terminal emulator configured at `.config/alacritty/alacritty.toml`
  - JetBrainsMono Nerd Font at size 11
  - Catppuccin Mocha colors (#1e1e2e background, #cdd6f4 foreground)
  - 95% opacity with 10px padding

- **Zsh**: Primary shell (`.zshrc`)
  - Plugins: zsh-autosuggestions, zsh-syntax-highlighting
  - FZF integration with Catppuccin colors
  - Starship prompt (configured at `.config/starship.toml`)
  - Key aliases: `vim`→`nvim`, `cat`→`bat`, `v`→`nvim`, `n`→`nnn`

### Text Editor
- **Neovim**: LazyVim distribution at `.config/nvim/`
  - Entry point: `init.lua` (bootstraps lazy.nvim)
  - Configuration structure:
    - `lua/config/`: Core config (keymaps, autocmds, options, lazy.lua)
    - `lua/plugins/`: Plugin specifications
  - Formatter: stylua configured at `stylua.toml`

### File Management & Utilities
- **nnn**: Terminal file manager
  - Configured with plugins (preview-tui, fzopen)
  - Trash support enabled
  - Custom color scheme

- **cmus**: Terminal music player
- **wiremix**: PipeWire TUI audio mixer (replaces pulsemixer)
- **bluetui**: Bluetooth TUI manager
- **impala**: Wi-Fi TUI manager (requires iwd)
- **pamixer**: CLI audio control (volume keys)
- **btop**: System monitor

## Key Sway Keybindings

- `Mod+Return`: Launch terminal (Alacritty)
- `Mod+Space`: Application launcher (Wofi) — matches Hyprland's `Super+Space`; `Mod+d` also still works
- `Mod+n`: nnn file manager
- `Mod+m`: cmus music player
- `Mod+Ctrl+a`: wiremix audio control — matches Hyprland's `Super+Ctrl+A`
- `Mod+Ctrl+b`: bluetui Bluetooth manager — matches Hyprland's `Super+Ctrl+B`
- `Mod+Ctrl+w`: impala Wi-Fi manager — matches Hyprland's `Super+Ctrl+W`
- `Mod+c`: Claude AI (Chromium)
- `Mod+w`: Kill window — matches Hyprland's `Super+W`; `Mod+Shift+q` also still works
- `Print`: Screenshot with grim/slurp

Keybinding convention (shared with this machine's Hyprland config): plain `Mod+letter` = window management, `Mod+Ctrl+letter` = system/TUI tools. Keep new Sway bindings aligned to this pattern so muscle memory transfers between machines.

## System Services

Auto-started via Sway config:
- pipewire & pipewire-pulse (audio)
- dunst (notifications)
- swayidle (screen lock/power management: 5min lock, 10min screen off)

## Environment Variables

Key environment variables (`.zshrc`):
- `WLR_NO_HARDWARE_CURSORS=1`: Required for proper cursor rendering
- `WLR_RENDERER=pixman`: Software rendering fallback
- `PATH=~/.npm-global/bin:$PATH`: npm global packages

## Color Scheme

Catppuccin Mocha is consistently applied across all tools:
- Base: #1e1e2e (background)
- Text: #cdd6f4 (foreground)
- Blue: #89b4fa (accents/focused)
- Mantle: #313244 (surfaces)
- Surface: #45475a (inactive elements)

## Modifying Configurations

When editing configuration files:
1. Sway config changes: Reload with `Mod+Shift+c` or run `swaymsg reload`
2. Waybar changes: Restart waybar process (`pkill waybar && waybar &`)
3. Alacritty config: Auto-reloaded on save
4. Neovim config: Use `:Lazy` command for plugin management
5. Shell config: Source with `source ~/.zshrc`

## Font Requirements

All configurations use **JetBrainsMono Nerd Font**. Ensure this is installed system-wide for consistent rendering across all applications.

---
> Source: [windseeker5/dotfiles](https://github.com/windseeker5/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
