## nixos-public

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a NixOS dotfiles repository using flakes for system configuration management. The main configuration lives in the `system/` directory with Home Manager integration for user-specific settings.

### Key Directories
- `system/` - Main NixOS system configuration
- `system/applications/` - Application-specific configurations (rofi, sunshine, etc.)
- `system/modules/` - Custom NixOS modules
- `system/nvim/` - Neovim configuration files
- `users/felipe/homeassistant/` - Home Assistant companion configuration

## Build Commands

### System Rebuild
```bash
# Primary rebuild command (also available as alias: rebuild)
sudo nixos-rebuild switch --flake ~/.dotfiles#default

# Test without switching (also available as alias: rebuild-test)
sudo nixos-rebuild test --flake ~/.dotfiles#default

# Build configuration only
sudo nixos-rebuild build --flake ~/.dotfiles#default

# Boot into new configuration on next reboot (also available as alias: rebuild-boot)
sudo nixos-rebuild boot --flake ~/.dotfiles#default
```

### Flake Management
```bash
# Update flake inputs (also available as alias: update-flake)
nix flake update ~/.dotfiles/system/

# Show flake info
nix flake show ~/.dotfiles/system/

# Check flake for issues
nix flake check ~/.dotfiles/system/
```

## Architecture Overview

### Configuration Files
- `flake.nix` - Defines inputs (nixpkgs, home-manager, zen-browser) and system configuration
- `configuration.nix` - Main system configuration including hardware, networking, services, users, and packages
- `home.nix` - Home Manager configuration for user 'felipe' including dotfiles, programs, and desktop settings
- `hardware-configuration.nix` - Hardware-specific configuration (auto-generated)

### Key Features Configured
- **Desktop Environment**: GNOME with X11 (Wayland disabled)
- **Power Management**: auto-cpufreq with performance/powersave profiles
- **Development Tools**: VSCode, Cursor, Git, Node.js 22, Python 3, Docker
- **Terminal Setup**: Kitty terminal with Zsh, Oh-My-Zsh, Starship prompt, Yazi file manager
- **Virtualization**: Docker, libvirtd with QEMU/KVM support
- **Remote Access**: Sunshine game streaming, Tailscale VPN, SSH
- **Applications**: Rofi launcher, various browsers (Brave, Chrome, Ungoogled Chromium)

### Home Manager Integration
Home Manager is configured as a NixOS module, managing:
- Shell configuration (Zsh with Oh-My-Zsh and Starship)
- Terminal emulator (Kitty with custom keybindings and theming)
- File manager (Yazi with custom configuration)
- Desktop environment customizations (GNOME extensions and keybindings)
- Git configuration
- Rofi application launcher with custom themes and scripts

### Rofi Configuration
Custom Rofi setup with:
- Multiple modes: applications, windows, SSH, emoji
- Custom keybindings via GNOME (Alt+n for launcher, Alt+w for windows, Alt+e for emoji)
- Custom themes and scripts in `system/applications/rofi/`
- Additional rofi scripts: brightness control, power management, system actions
- Desktop entries for easy access

### Services and Daemons
- **hacompanion**: Home Assistant companion service
- **Sunshine**: Game streaming server with hardware acceleration
- **Tailscale**: VPN service
- **auto-cpufreq**: Dynamic CPU frequency scaling
- **thermald**: Thermal management

## Development Environment

### Languages and Tools
- **Node.js**: Version 22 installed system-wide
- **Python**: Python 3 with development packages available via nix-shell
- **Git**: Configured with user details and helpful aliases
- **Docker**: Enabled with user in docker group
- **Shell**: Zsh with Oh-My-Zsh, Starship prompt, and useful aliases

### Shell Configuration
- **Terminal**: Kitty with Catppuccin Mocha theme
- **File Manager**: Yazi with custom keybindings and configuration
- **Navigation**: Zoxide for smart directory jumping
- **Aliases**: ls → eza, cd → z (zoxide), pbcopy/pbpaste for clipboard

### Tmux Configuration

Terminal multiplexer with advanced session management and custom plugins. See [docs/tmux.md](docs/tmux.md) for complete documentation.

**Quick Start:**
- Prefix: `Ctrl-Space` (primary) or `Ctrl-b` (fallback)
- Catppuccin Mocha theme with powerline separators
- TPM plugin manager for enhanced features
- Key plugins: sessionx, thumbs, floax, tmux-fzf

**Essential Keybindings:**
- `Prefix + o` - Session manager
- `Prefix + Space` - Copy hints
- `Prefix + F` - Floating terminal
- `Ctrl+h/j/k/l` - Navigate panes (vim-aware)

### Fonts
- JetBrains Mono (with Nerd Font support)
- SF Pro Display set as system default
- Fira Code, Cascadia Code, IBM Plex available

## Power Management

Integrated CPU frequency and power profile control with multiple access methods:

### GUI Access
- **GNOME Settings**: Settings → Power → Power Mode (Power Saver, Balanced, Performance)
- **GNOME Top Bar**: Power profiles shown in system menu (when power-profiles-daemon is active)
- **Extensions**: System monitor extensions show current CPU governor

### Keyboard Shortcuts
- `Super+Alt+1` - Power Saver mode (maximum battery life)
- `Super+Alt+2` - Balanced mode (optimal performance/power balance)
- `Super+Alt+3` - Performance mode (maximum performance)

### Terminal Commands
- `power-saver` - Switch to power saver mode
- `balanced` - Switch to balanced mode  
- `performance` - Switch to performance mode
- `power-status` - Show current power profile
- `power-list` - List all available power profiles

### Graceful Shutdown
To prevent Chrome from showing "restore tabs" after shutdown/reboot:
- **Extended timeouts**: DefaultTimeoutStopSec=300s gives apps 5 minutes to close gracefully
- **Configured in**: systemd.extraConfig in main configuration
- **Alternative**: Manually close Chrome before shutdown for clean state

### Hibernation Requirements
Hibernate and hybrid sleep require swap space equal to or larger than RAM size:
- **Current setup**: 62GB RAM, 39GB swap (insufficient)
- **To enable hibernation**:
  1. Create swap file: `sudo fallocate -l 64G /swapfile`
  2. Set permissions: `sudo chmod 600 /swapfile` 
  3. Format: `sudo mkswap /swapfile`
  4. Enable: `sudo swapon /swapfile`
  5. Add to NixOS config: `boot.resumeDevice = "/swapfile";`

### Power Profiles Explained
- **Power Saver**: `powersave` CPU governor, reduced frequencies, maximum battery life
- **Balanced**: `ondemand` CPU governor, adaptive frequencies, good balance
- **Performance**: `performance` CPU governor, maximum frequencies, best responsiveness

## Window Management & Tiling

Advanced window tiling with i3-like keyboard shortcuts:

### Tiling Shortcuts
- `Super+Left` - Tile window to left half
- `Super+Right` - Tile window to right half  
- `Super+Up` - Tile window to top half
- `Super+Down` - Tile window to bottom half

### Features
- **Tiling Assistant Extension**: Enhanced tiling with quarter-screen, fancy zones
- **Smart Gaps**: Automatic gaps between tiled windows  
- **Multi-Monitor**: Tiling works across multiple monitors
- **Window Layouts**: Supports various layouts and window arrangements

## System Monitoring

Comprehensive monitoring with multiple specialized tools:

### Available Commands
- `sysmon` - Interactive system monitor (btop)
- `sysinfo` - System information overview (neofetch)
- `temps` - Hardware temperature monitoring
- `processes` - Process tree view with details
- `disk-usage` - Disk space overview with colors
- `disk-health` - SMART disk health status
- `network-usage` - Network usage by process
- `io-monitor` - Real-time I/O activity

### Monitoring Categories
- **System Overview**: CPU, memory, load, uptime
- **Hardware**: Temperatures, disk health, GPU usage
- **Performance**: Process usage, I/O activity, network traffic
- **Storage**: Disk space, file system usage, SMART status
- **Network**: Bandwidth usage, connections, per-process usage

### Key Features
- **Zero Overhead**: Tools run only when needed
- **Detailed Information**: Comprehensive system insights
- **No Web Dependencies**: All terminal-based for reliability
- **Hardware Monitoring**: Temperature and health sensors

## Automatic Updates

Safe, automated system maintenance:

### Update Schedule
- **System Updates**: Daily at 4:00 AM (randomized ±45min)
- **Garbage Collection**: Weekly cleanup of old generations
- **Store Optimization**: Weekly deduplication and compression

### Safety Features
- **No Auto-Reboot**: Updates require manual reboot
- **Rollback Available**: Previous generations kept for 30 days
- **Build Logs**: Detailed logs for troubleshooting
- **Fallback**: Uses binary substitutes when available

## Theme Switching

Custom theme switching scripts are available in `system/scripts/`:

### Available Commands
- `dark-theme` - Switch GNOME to dark theme
- `light-theme` - Switch GNOME to light theme

## Screenshots

Flameshot is configured as the default screenshot application:

### Usage
- **Print Screen** - Launch Flameshot GUI for area selection and annotation
- **Manual launch** - Run `flameshot gui` from terminal

### Features
- Area selection with drag and drop
- Built-in annotation tools (arrows, text, shapes, blur)
- Direct upload to image hosting services
- Clipboard integration
- Pin screenshots to screen

## Common Tasks

### Making Configuration Changes
1. Edit the appropriate .nix file (see quick edit aliases below)
2. Test changes: `rebuild-test` or `sudo nixos-rebuild test --flake ~/.dotfiles#default`
3. If successful, apply: `rebuild` or `sudo nixos-rebuild switch --flake ~/.dotfiles#default`
4. Home Manager changes are applied automatically during system rebuild

### Quick Edit Aliases
- `edit-config` - Edit main system configuration
- `edit-home` - Edit Home Manager configuration
- `dot` - Navigate to dotfiles directory

### Troubleshooting
- Check system logs: `journalctl -xeu systemd-service-name`
- View build logs: Add `--show-trace` to nixos-rebuild commands
- Roll back: `sudo nixos-rebuild switch --rollback`

## Security and Hardware
- **Firewall**: Configured with specific ports for Sunshine and development
- **Bluetooth**: Enabled
- **Audio**: PipeWire with ALSA and PulseAudio compatibility
- **Graphics**: Intel GPU tools, Mesa, Vulkan support
- **Power**: Optimized for laptop use with SSD trim and zram swap

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LFTPadilla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
