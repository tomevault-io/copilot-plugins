## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal dotfiles repository for macOS containing configuration files and scripts for various development tools and applications. The repository uses a bare git repository approach for dotfiles management.

## Key Tools and Applications

### Terminal and Shell
- **Ghostty**: Modern terminal emulator with glass effects and theme support
  - Config: `ghostty/.config/ghostty/config`
  - Theme: Uses dark:nightfox,light:catppuccin-latte.conf
  - Font: JetBrains Mono, size 15
- **Zsh**: Shell with Oh My Zsh framework
  - Config: `zsh/.zshrc`
  - Theme: af-magic
  - Git plugin enabled

### Development Environment
- **Vim**: Text editor with minimal plugin setup
  - Config: `vim/.vimrc`
  - Plugin manager: vim-plug
  - Theme: catppuccin-latte
  - Cursor shapes configured for Ghostty terminal
- **SPF (Superfile)**: Terminal file manager
  - Config: `spf/.spf.toml`
  - Theme: catppuccin-latte
  - Editor: vim

### Applications
- **MacroWhisper**: Voice-to-text application
  - Config: `macrowhisper/.config/macrowhisper/`
- **MyPaint**: Digital painting application
  - Config: `mypaint/.config/mypaint/`
- **iTerm2**: Alternative terminal (legacy configuration)
  - Config: `iterm/Default.json` and `iterm/Untitled.itermkeymap`

## Git Configuration Management

This repository uses the bare repository approach for dotfiles management:

```bash
# Main command alias for managing dotfiles
alias config='/usr/bin/git --git-dir=/Users/filipmellqvist/.dotfiles/ --work-tree=/Users/filipmellqvist'

# Usage examples:
config status
config add .vimrc
config commit -m "update vim config"
config push
```

## Development Aliases and Scripts

### Vim Keymap Management
Dynamic keymap switching system supporting both terminal Vim and VSCode/Cursor:
- `vimtoggle`: Toggle between default (HJKL) and custom (JKLÖ) keymaps
- `vimkeys status`: Show current keymap status
- `vimkeys default`: Force default HJKL mode
- `vimkeys custom`: Force custom JKLÖ mode

**Integration Features:**
- **Real-time hot-reload**: All running Vim instances switch automatically
- **VSCode/Cursor support**: Dynamically updates VSCodeVim extension settings
- **State persistence**: Remembers keymap choice across sessions
- **FastScripts integration**: System-wide keyboard shortcut access
- **Visual feedback**: Status line indicators and macOS notifications

**Key Files:**
- Main script: `my_scripts/.script/vim-keymap-toggle.sh`
- VSCode manager: `my_scripts/.script/vscode-vim-keymap-manager.sh`
- State file: `vim/.vim/keymap_state`
- FastScripts: `~/Library/Scripts/Toggle Vim Keymaps.sh`
- Backups: `vim/.vim/vscode-backups/`

### Port Management
The zsh configuration includes comprehensive port management aliases:
- `kill3000`, `kill3001`, `kill3002`, `kill3003`, `kill5555`: Kill processes on specific ports
- `3000`, `3001`, `3002`, `3003`: Kill port processes and start development servers
- `npm3001`, `npm3002`, `npm3003`: Start npm dev servers on specific ports
- `5555`: Kill port 5555 and start Prisma Studio
- `killnpmall`: Kill all npm development ports (3000-3002)

### Application Shortcuts
- `sp`: Start SPF file manager with config
- `mw`: Start MacroWhisper
- `tm`: Start task-master

### Development Environment
- OpenJDK 11 configured at `/opt/homebrew/opt/openjdk@11/bin`
- Bun runtime configured
- Node.js development focused with TypeScript script execution

## Script Collection

The `my_scripts/.script/` directory contains various utility scripts:

### Certificate Management
- `ca-cert-creator/`: Scripts for creating CA certificates and SSL certificates
  - `creator.sh`, `creator_noconf.sh`, `creator_CA.sh`

### Audio Configuration
- `pulseaudio/`: PulseAudio configuration scripts
- `pulseaudio_volume.sh`: Volume control script
- `pulseaudio_sink_switch.sh`: Audio sink switching
- `sony_wh-1000xm3.sh`: Sony headphones configuration

### System Utilities
- `wacom-config.sh`, `wacom_precision_toggler.sh`: Wacom tablet configuration
- `feh_random_wallpaper.sh`, `feh_95_wallpaper.sh`: Wallpaper management
- `keyboard_layouts/`: Keyboard layout configurations
- `miniscripts/`: Collection of small utility scripts

## Configuration Patterns

### Theme Consistency
All applications use consistent theming:
- **Light theme**: catppuccin-latte
- **Dark theme**: catppuccin-mocha/nightfox
- **Font**: JetBrains Mono across all applications

### Editor Integration
- Vim configured with terminal cursor shape integration
- Consistent editor (vim) across file managers and configuration tools
- System clipboard integration enabled

## File Structure

```
.
├── ghostty/          # Terminal emulator config
├── vim/              # Vim editor configuration and plugins
├── zsh/              # Zsh shell configuration
├── spf/              # Superfile manager config
├── macrowhisper/     # Voice-to-text app config
├── mypaint/          # Digital painting app config
├── iterm/            # Legacy iTerm2 configuration
├── my_scripts/       # Collection of utility scripts
└── MW Macros.kmmacros # Keyboard Maestro macros
```

## Best Practices

When modifying configurations:
1. Test changes in the target application before committing
2. Use the `config` alias for all git operations in this repository
3. Maintain theme consistency across applications
4. Back up existing configurations before major changes
5. Document any new aliases or scripts added to the zsh configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cretiq) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
