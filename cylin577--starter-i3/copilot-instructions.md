## starter-i3

> This directory contains `starter-i3`, a set of configuration files and a setup script designed to provide a visually appealing and functional i3 window manager setup with minimal effort. It's aimed at new i3 users who want a good-looking and working configuration out of the box.

# GEMINI.md - starter-i3

## Project Overview

This directory contains `starter-i3`, a set of configuration files and a setup script designed to provide a visually appealing and functional i3 window manager setup with minimal effort. It's aimed at new i3 users who want a good-looking and working configuration out of the box.

The setup includes:

*   **i3:** A tiling window manager. The configuration is in `i3/config`.
*   **Polybar:** A status bar that displays information about workspaces, the current window, date/time, and system metrics. The configuration is in `polybar/config.ini`.
*   **Picom:** A compositor for visual effects like transparency and shadows. The configuration is in `picom.conf`.
*   **Alacritty:** A fast, cross-platform, OpenGL terminal emulator.
*   **Rofi:** A window switcher, application launcher, and dmenu replacement.
*   **Feh:** An image viewer used here to set the wallpaper.
*   **JetBrains Mono:** The primary font used in the configuration.

## Building and Running

The project uses a shell script, `geti3.sh`, to automate the installation and configuration process. The script performs the following actions:

1.  **Detects available package managers** (`apt` or `pacman`). If both are found, it will prompt the user to choose.
2.  **Installs necessary packages** using the appropriate package manager (`apt` or `pacman`).
3.  **Backs up existing configuration files** in `~/.config/i3` and `~/.config/picom.conf`.
5.  **Copies the new configuration files** to `~/.config/` (including i3, picom, and polybar configurations).
5.  **Sets the wallpaper** by copying `wallpaper.jpg` to `~/Pictures/`.
6.  **Ensures `~/.xinitrc` and `~/.xsession` are present** to start i3 correctly.

To run the setup script, execute the following command in the project's root directory:

```bash
sudo bash ./geti3.sh
```

## Development Conventions

### i3 Configuration (`i3/config`)

*   **Mod Key:** The `$mod` key is set to `Mod4` (the Windows key).
*   **Font:** `JetBrains Mono` is used for the i3 status bar and window titles.
*   **Startup Applications:**
    *   `feh` sets the wallpaper.
    *   `picom` is started for compositing.
    *   `autostart.sh` is executed for other startup items.
    *   `fcitx5` is started for input method support.
    *   `polybar` is launched.
*   **Keybindings:**
    *   `$mod+Return`: Launch Alacritty terminal.
    *   `$mod+d`: Launch Rofi for application launching.
    *   `$mod+Shift+q`: Kill the focused window.
    *   `$mod+h/j/k/l`: Move focus left/down/up/right.
    *   `$mod+Shift+h/j/k/l`: Move the focused window.
    *   `$mod+1/2/3/4`: Switch to the specified workspace.
    *   `$mod+Shift+c`: Reload i3 configuration.
    *   `$mod+Shift+r`: Restart i3.
    *   `$mod+Shift+e`: Display the exit menu.
    *   `$mod+Escape`: Enter system mode for lock, logout, reboot, shutdown.

### Polybar Configuration (`polybar/config.ini`)

*   **Appearance:** The bar is configured with a dark background (`#33312c`) and white foreground (`#ffffff`).
*   **Modules:**
    *   **Left:** `i3` (workspaces), `xwindow` (current window title).
    *   **Center:** `date` (current date and time).
    *   **Right:** `github` (notifications), `temperature` (CPU temp), `tray` (system tray).
*   **Customization:** The configuration file has a commented-out section for a `battery` module. To enable it, you need to add `"battery"` to the `modules-right` list and configure the `[module/battery]` section with your battery and adapter names.

### Picom Configuration (`picom.conf`)

This file contains settings for the Picom compositor, which controls transparency, shadows, and fading effects for windows and menus. The default configuration provides a subtle and modern look.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cylin577) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
