## omarchy-ravenwood-theme

> This project provides a set of themes (Dark and Light) specifically designed for the Omarchy environment, featuring custom color schemes, backgrounds, and integration for various developer tools.

# Ravenwood Themes for Omarchy

This project provides a set of themes (Dark and Light) specifically designed for the Omarchy environment, featuring custom color schemes, backgrounds, and integration for various developer tools.

## Project Overview

*   **Type:** Theme Collection / Configuration Project
*   **Target:** [Omarchy](https://omarchy.org/)
*   **Key Features:**
    *   **Ravenwood (Dark):** Deep forest green and dark grey (`#1a1f1c`) with emerald accents (`#4ade80`).
    *   **Ravenwood Light:** A lighter variant (`#fdf6e3`) with blue/teal accents.
    *   **Application Support:** Includes configurations for Neovim, VS Code, and btop.
    *   **Dynamic Theme Switching:** A systemd-based service to automatically toggle between dark and light modes based on the time of day.

## Directory Structure

*   `ravenwood/`: Contains the dark theme assets and configurations.
    *   `colors.toml`: Core color definitions.
    *   `backgrounds/`: Desktop wallpapers.
    *   `scripts/`: Contains the dynamic theme switching logic and systemd units.
    *   `neovim.lua`, `vscode.json`, `btop.theme`: Application-specific theme files.
*   `ravenwood-light/`: Contains the light theme assets and configurations.
*   `install.sh`: The primary installation script for deploying themes and setting up services.

## Installation and Usage

### Installation
The themes should be installed using the provided `install.sh` script:
```bash
./install.sh
```
This script performs the following:
1.  Copies both `ravenwood` and `ravenwood-light` to `~/.config/omarchy/themes/`.
2.  Cleans up legacy backgrounds.
3.  Optionally installs and enables a systemd user service for dynamic theme switching.

### Applying Themes Manually
Use the `omarchy` CLI to switch themes:
```bash
omarchy-theme-set ravenwood
omarchy-theme-set ravenwood-light
```

### Dynamic Theme Switching
The dynamic switcher is managed via systemd:
*   **Service:** `omarchy-dynamic-theme.service`
*   **Timer:** `omarchy-dynamic-theme.timer`
*   **Logic:** Defined in `~/.config/omarchy/themes/ravenwood/scripts/dynamic-theme.sh`. It defaults to Light mode between 7 AM and 7 PM.

## Development Conventions

*   **Colors:** Color palettes are defined in `colors.toml` using TOML format.
*   **Tooling:** Themes for external tools (Neovim, VS Code) should be kept in sync with the core `colors.toml` values.
*   **Scripts:** Bash scripts use `set -e` for safety and rely on `omarchy` CLI commands (`omarchy-theme-current`, `omarchy-theme-set`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raythurman2386)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raythurman2386)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
