## nix-hyprland-desktop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build and Format
- `nix develop` - Enter development shell with all dependencies
- `nix fmt` - Format all Nix files using nixfmt-rfc-style
- `nix flake check` - Check flake validity and formatting

### Testing Waybar Configuration
```bash
cd /path/to/nix-hyprland-desktop/dotfiles/waybar
./build-config.sh  # Rebuild modular waybar config
waybar -c config -s style.css --log-level debug  # Test waybar changes
```


## Architecture Overview

This is a **standalone NixOS flake** that provides a complete Hyprland desktop environment configuration. It's designed to be imported into any NixOS system as a module.

### Core Components

1. **Flake Structure** (`flake.nix`):
   - Exports `nixosModules.hyprland-desktop` for importing into NixOS configs
   - Provides development shell with formatting tools
   - Uses nixpkgs 25.05 with Hyprland flake input

2. **Main Module** (`modules/hyprland-desktop.nix`):
   - Comprehensive Hyprland desktop setup with all dependencies
   - Configures services: PipeWire, NetworkManager, gnome-keyring, greetd
   - Deploys dotfiles to `/etc/xdg` for system-wide availability with user override support
   - Includes keyring unlock utility and XDG config path resolution tools

3. **Dotfiles Structure** (`dotfiles/`):
   - **hypr/**: Hyprland compositor config with scripts
   - **waybar/**: Modular status bar config system with working taskbar (see Waybar section below)
   - **wofi/**: Application launcher styling
   - **swaync/**: Notification daemon theming (Kartoza branded) - see important layer config note below
   - **fuzzel/**: Additional launcher utilities

### Swaync Layer Configuration

**IMPORTANT**: The swaync `config.json` uses `"layer": "top"` instead of `"overlay"`. This is intentional:

- `"overlay"` layer creates an invisible input-capturing region on the right side of the screen
- This blocks mouse clicks on window title bar buttons (close, minimize, maximize) in the upper-right
- `"top"` layer properly handles input passthrough when the notification panel is closed
- Notifications still appear above windows but don't block interaction with window controls

### Waybar Modular Configuration System

The waybar config uses a **unique modular approach** for easier maintenance:

- `config.d/*.json` - Individual feature modules (base, widgets, custom modules)
- `build-config.sh` - Merges JSON files using `jq` into final `config`
- Numbering system: `00-` (base), `10-` (core modules), `90-` (UI widgets)
- Build process automatically includes `wlr/taskbar` and `wlr/workspaces` modules for Hyprland
- Taskbar now works with proper `hyprctl` commands for window management

### Theme Integration

- Expects `config.kartoza.theme.iconTheme.name` from importing flake (defaults to Papirus)
- Kartoza branding with custom logos and color schemes
- Orange accent color (`#DF9E2F`) for active window borders in Hyprland

### Keyboard Layout Configuration

The module provides configurable keyboard layouts with intelligent switching:

- **Default**: `["us", "pt"]` (US English, Portuguese)
- **Customizable**: Set any list of layouts via `keyboardLayouts` option
- **Smart Toggle**: Waybar script automatically reads layouts from Hyprland config
- **Alt+Shift**: Hardware toggle between configured layouts
- **Display Names**: Automatic conversion (us→EN, de→DE, fr→FR, pt→PT, etc.)

Example configuration:
```nix
kartoza.hyprland-desktop = {
  enable = true;
  keyboardLayouts = [ "us" "de" "fr" ];  # US, German, French
};
```

### Wallpaper Configuration

The module provides unified wallpaper management across desktop and lock screen:

- **Default**: `/etc/wallpapers/kartoza.png` (Kartoza branded wallpaper)
- **Configurable**: Set custom wallpaper path via `wallpaper` option
- **Unified**: Same wallpaper used for desktop background (swww) and lock screen (swaylock)
- **Styled Lock Screen**: Swaylock overlay with Kartoza colors, blur effects, clock, and indicators

Example configuration:
```nix
kartoza.hyprland-desktop = {
  enable = true;
  wallpaper = "/home/user/Pictures/custom-wallpaper.jpg";  # Custom wallpaper
};
```

### User Configuration Override Support

The module follows XDG Base Directory Specification for configuration management:

- **System configs**: `/etc/xdg/hypr/`, `/etc/xdg/waybar/`, etc. (provided by module)
- **User overrides**: `~/.config/hypr/`, `~/.config/waybar/`, etc. (user customizations)
- **Resolution order**: User configs in `~/.config/` take precedence over system configs in `/etc/xdg/`

#### XDG Config Tools

- `xdg-config-path` - Simple path helper for shell scripts
- PATH includes both `~/.config/*/scripts` and `/etc/xdg/*/scripts` (user scripts first)
- All scripts use hard-coded `/etc/xdg/` paths for consistent configuration resolution

#### Overriding Configuration

Users can override any system configuration by copying files to their home directory:

```bash
# Override hyprland config
cp /etc/xdg/hypr/hyprland.conf ~/.config/hypr/

# Override waybar config
mkdir -p ~/.config/waybar
cp /etc/xdg/waybar/config ~/.config/waybar/
cp /etc/xdg/waybar/style.css ~/.config/waybar/

# Override individual waybar modules
mkdir -p ~/.config/waybar/config.d
cp -r /etc/xdg/waybar/config.d/* ~/.config/waybar/config.d/

# Override swaync notification settings
mkdir -p ~/.config/swaync
cp /etc/xdg/swaync/config.json ~/.config/swaync/
cp /etc/xdg/swaync/style.css ~/.config/swaync/
```

All applications and scripts will automatically use the user's configuration if present.

### Workspace Management System

The module provides a comprehensive workspace management system with named workspaces and fuzzel-based switching:

#### Features

- **Named Workspaces**: Each workspace can have a custom name (Browser, Chat, Terminal, etc.)
- **Fuzzel Switcher**: Beautiful graphical workspace selector with current workspace indicator
- **Waybar Integration**: Clickable workspace widget showing current workspace, plus working taskbar
- **Change Tracking**: Automatic logging and notifications when switching workspaces
- **User Customizable**: Override workspace names via user configuration
- **Hyprctl Integration**: Uses `hyprctl` for reliable workspace switching and status

#### Default Workspace Layout (3×3 Grid)

```
┌─────────────┬─────────────┬─────────────┐
│ 0: Browser  │ 1: Chat     │ 2: Terminal │
├─────────────┼─────────────┼─────────────┤
│ 3: Project1 │ 4: Project2 │ 5: Media    │
├─────────────┼─────────────┼─────────────┤
│ 6: Documents│ 7: Games    │ 8: System   │
└─────────────┴─────────────┴─────────────┘
```

#### Keyboard Shortcuts

- **`Super + S`** - Open fuzzel workspace switcher
- **`Super + 1-9`** - Switch directly to workspace 1-9
- **`Ctrl + Super + Arrow Keys`** - Navigate workspace grid
- **`Super + Shift + 1-9`** - Move current window to workspace
- **`Super + Shift + Ctrl + Arrows`** - Move window in workspace grid

#### Waybar Widget

The waybar includes a workspace widget that:
- Shows current workspace number and name (e.g., "1: Browser")
- Click to open fuzzel workspace switcher
- Updates automatically when workspace changes
- Styled with Kartoza orange accent colors

#### Managing Workspace Names

```bash
# Show current workspace
workspace-names.sh current

# List all workspace names  
workspace-names.sh list

# Rename a workspace
workspace-names.sh set 0 "Web Browser"
workspace-names.sh set 3 "Development"

# Get specific workspace name
workspace-names.sh get 1
```

#### Customizing Workspace Names

```bash
# Copy system config to user location
cp /etc/xdg/hypr/workspace-names.conf ~/.config/hypr/

# Edit workspace names
# Format: workspace_number=workspace_name
echo "0=My Browser" >> ~/.config/hypr/workspace-names.conf
echo "1=Slack" >> ~/.config/hypr/workspace-names.conf
```

#### Workspace Scripts

- `workspace-switcher.sh` - Fuzzel-based workspace selector
- `workspace-names.sh` - Workspace name management utility
- `workspace-changed.sh` - Hook called when workspace changes
- `workspace-display.sh` - Waybar widget for current workspace display

### Key Scripts and Utilities

- `unlock-keyring` - GUI keyring unlock at login using zenity
- `hypr/scripts/` - Workspace management, recording toggles, browser detection (updated for Hyprland)
- `waybar/scripts/` - Status monitoring (temperature, power, notifications, workspace display)

## Development Workflow

1. **Making Config Changes**: Edit files in `dotfiles/` subdirectories
2. **Waybar Changes**: Use modular system in `config.d/`, run `build-config.sh`
3. **Testing**: Use `nix develop` shell, test waybar with live reload
4. **Module Integration**: Changes are deployed via NixOS rebuild when module is imported

## Integration Notes

- Module configures complete Wayland environment
- Uses greetd with regreet greeter for display management with Kartoza theming
- Includes screen sharing support via xdg-desktop-portal-hyprland
- PAM integration for keyring unlock on login and screen unlock
- Environment variables set for proper Wayland app compatibility
- Windows spawn in floating mode by default (can be toggled to tiling with Super+F)

## External Application Integrations

The waybar configuration includes widgets that integrate with optional external applications. These applications must be installed separately in your NixOS configuration.

### Kartoza Timesheet App

Repository: `github:kartoza/go-timesheets-go`

The timesheet widget shows current time tracking status:
- **Widget Location**: Left panel, shows timer icon with elapsed time or idle status
- **Click Action**: Opens timesheet TUI in kitty terminal
- **Status Command**: `kartoza-timesheet status` (returns waybar-compatible JSON)

Waybar configuration (`90-custom-timesheet.json`):
```json
{
  "custom/timesheet": {
    "exec": "kartoza-timesheet status",
    "return-type": "json",
    "interval": 60,
    "on-click": "kitty kartoza-timesheet"
  }
}
```

### Kartoza Video Processor

Repository: `github:kartoza/kartoza-video-processor`

The recorder widget shows screen recording status:
- **Widget Location**: Left panel, shows video camera icon
- **Click Action**: Toggles recording on/off
- **Status Command**: `kartoza-video-processor status --waybar` (returns waybar-compatible JSON)
- **Keybinding**: `Ctrl+6` toggles recording

Waybar configuration (`90-custom-recorder.json`):
```json
{
  "custom/recorder": {
    "exec": "kartoza-video-processor status --waybar",
    "return-type": "json",
    "interval": 2,
    "on-click": "kartoza-video-processor toggle"
  }
}
```

**Fallback**: If `kartoza-video-processor` is not installed, the `recording-status.sh` script falls back to legacy PID-based detection for `wl-screenrec`/`pw-record`/`webcam-recorder` processes.
- whenever you update this repo, always validate is afterwards using:

hyprland --verify-config --config dotfiles/hypr/hyprland.conf

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kartoza)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kartoza)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
