## dotfiles

> This is a **chezmoi** source directory used for managing user configuration files (dotfiles) on a Linux system. It centralizes the configuration for the shell, window manager, editor, and other utilities, allowing for consistent setups across multiple machines (referenced by hostname tags like `owlenlap01`, `owlenpc00`).

# Project Overview

This is a **chezmoi** source directory used for managing user configuration files (dotfiles) on a Linux system. It centralizes the configuration for the shell, window manager, editor, and other utilities, allowing for consistent setups across multiple machines (referenced by hostname tags like `owlenlap01`, `owlenpc00`).

## Key Components

*   **Dotfile Manager:** [chezmoi](https://www.chezmoi.io/)
*   **Window Manager:** [Hyprland](https://hyprland.org/) (Modular configuration)
*   **Shell:** [Zsh](https://www.zsh.org/) managed with [Zim](https://zimfw.sh/) and themed with [Powerlevel10k](https://github.com/romkatv/powerlevel10k).
*   **Editor:** [Neovim](https://neovim.io/) configured with Lua and [lazy.nvim](https://github.com/folke/lazy.nvim).
*   **Status Bar:** [Waybar](https://github.com/Alexays/Waybar)
*   **Terminal Emulators:** [Kitty](https://sw.kovidgoyal.net/kitty/), [Alacritty](https://alacritty.org/)
*   **Other:** Tmux, Mpv, Dunst/SwayNC, Fuzzel/Rofi equivalent.

## Directory Structure

The directory structure follows standard `chezmoi` conventions:

*   **`dot_config/`**: Maps to `~/.config/`. Contains the bulk of application configurations.
    *   **`hypr/`**: Hyprland configuration. `hyprland.conf` is the entry point, sourcing files from `hyprland.d/`. Host-specific overrides exist (e.g., `hyprpaper.conf##hostname.owlenlap01`).
    *   **`nvim/`**: Neovim configuration. `init.lua` bootstraps `lazy.nvim`.
    *   **`zsh/`**: Zsh configuration, including `dot_zshrc` (mapped to `~/.config/zsh/.zshrc`).
    *   **`waybar/`**: Waybar configuration and styles.
*   **`dot_local/`**: Maps to `~/.local/`. Contains scripts and local data.
    *   **`dot_local/share/docs/`**: Local docs area populated by git submodules using the `external_` attribute (renders to `~/.local/share/docs/`).
*   **`.chezmoiscripts/`**: Contains scripts that run when `chezmoi apply` is executed (e.g., `run_onchange_...`).
*   **`dot_profile.tmpl`**: A template that renders to `~/.profile`, handling environment variables. It uses tags (e.g., `dev`, `desktop`) to conditionally include configurations.

## Documentation

*   **Chezmoi docs:** `dot_local/share/docs/external_chezmoi-docs` (sparse checkout `assets/chezmoi.io`) renders to `~/.local/share/docs/chezmoi-docs`.
*   **Hyprland wiki:** `dot_local/share/docs/external_hyprland.wiki` (sparse checkout `content`) renders to `~/.local/share/docs/hyprland.wiki`.
*   **Waybar wiki:** `dot_local/share/docs/external_waybar.wiki` renders to `~/.local/share/docs/waybar.wiki`.
*   **Waybar man pages:** Not stored in the repo; use `man 5 waybar` and `man 5 waybar-<module>`.
*   **Ghostty docs:** Available locally at `/usr/share/ghostty`.

## Usage

This directory is the **source of truth**. Do not edit files directly in `~/.config` if they are managed here; your changes will be overwritten.

### Common Commands

*   **Apply Changes:**
    ```bash
    chezmoi apply
    ```
    Renders templates and copies files to the target destination (usually `$HOME`).

*   **Preview Changes:**
    ```bash
    chezmoi diff
    ```
    Shows what will change in the destination directory if `chezmoi apply` is run.

*   **Edit a File:**
    ```bash
    chezmoi edit <file_path>
    ```
    Opens the source file for editing in your `$EDITOR`.

*   **Check Status:**
    ```bash
    chezmoi status
    ```

### Templates and Logic

Files ending in `.tmpl` are Go templates. They support logic based on `chezmoi` data.
*   **Variables:** `{{ .chezmoi.homeDir }}` represents the user's home directory.
*   **Tags:** Logic like `{{- if (index $tags "dev") }}` is used to enable features based on the machine's role (e.g., development tools, desktop-specific GPU settings).

## Development Conventions

*   **Hyprland:** Keep configuration modular. Add new settings to the appropriate file in `dot_config/hypr/hyprland.d/` rather than cluttering `hyprland.conf`.
*   **Neovim:** Configuration is split into `lazy_setup.lua` (plugins) and `polish.lua` (general settings).
*   **Zsh:** Custom aliases should go in `dot_config/zsh/aliases.zsh`.
*   **Naming:** Use `dot_` to represent a dot prefix (e.g., `dot_config` -> `.config`). Use `executable_` to mark files as executable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mpuchstein) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
