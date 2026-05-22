## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a cross-platform dotfiles repository designed for consistent development environments on Mac (Aerospace) and Linux (Hyprland). The repository uses GNU Stow for symlink management and includes unified keybindings across both platforms.

## Common Commands

### Installation and Setup
- **Initial installation**: `./install.sh` - Installs all dotfiles and creates backups
- **Manual installation**: `stow -d ~/dotfiles/common -t ~/.config nvim tmux zsh`
- **Platform-specific install**: 
  - Mac: `stow -d ~/dotfiles/mac -t ~/.config aerospace`
  - Linux: `stow -d ~/dotfiles/linux -t ~/.config hypr waybar`

### Sync Operations
- **Commit and push to git**: `./sync.sh`
- **Push to remote Linux machine**: `./sync.sh push`
- **Pull Linux configs from remote**: `./sync.sh pull`

### Development Environment
- **Launch development session**: `dev` command creates/attaches to tmux session with nexus and nexus-cdk windows
- **Project navigation**: `p` command uses fzf to navigate to projects in ~/Developer
- **Process management**: `pkill` command with fzf selection
- **Git log viewer**: `logg` command with interactive preview and GitHub integration

### Shell Configuration
- **AWS profile management**: 
  - `asp <profile>` - Set AWS profile
  - `asp` - Clear AWS profile  
  - `asr <region>` - Set AWS region
  - `alp` - List available profiles
- **Starship prompt**: Configured and initialized in zshrc

## Architecture

### Directory Structure
```
dotfiles/
├── common/          # Cross-platform configurations
│   ├── git/        # Git config with delta for diffs
│   ├── nvim/       # LazyVim configuration
│   ├── tmux/       # Tmux with custom prefix (Ctrl+S)
│   ├── zsh/        # Shell configs with modular files
│   ├── atuin/      # Shell history sync
│   └── fzf/        # Fuzzy finder config
├── mac/            # Mac-specific (Aerospace)
├── linux/          # Linux-specific (Hyprland/Waybar)
├── install.sh      # Cross-platform installer
└── sync.sh         # Remote sync and git operations
```

### Key Configuration Details

#### Tmux Configuration
- **Prefix key**: Changed from Ctrl+B to Ctrl+S
- **Plugin manager**: TPM with plugins in ~/.config/tmux/plugins/
- **Key themes**: Supports both catppuccin-tmux and tokyo-night-tmux
- **Vim integration**: seamless pane navigation with vim-tmux-navigator
- **Custom keybindings**: `\` for horizontal split, `-` for vertical split

#### Neovim Configuration  
- **Distribution**: LazyVim starter template
- **Location**: ~/.config/nvim/ (symlinked from common/nvim/)
- **Plugin management**: Lazy.nvim with custom plugin configurations

#### Zsh Configuration
- **Modular structure**: Multiple .zsh files sourced from common/zsh/
- **Key modules**:
  - `aws.zsh`: AWS profile and region management functions
  - `cli_tools.zsh`: Project navigation, process management, git tools
  - `tmux.zsh`: Development environment automation
  - `fluree_commands.zsh`: Fluree-specific tooling

#### Cross-Platform Window Management
- **Unified keybindings**: Documented in KEYBINDINGS.md
- **Primary modifier**: Cmd (Mac) / Super (Linux)
- **Navigation**: Vim-style h/j/k/l across tmux, neovim, and window managers
- **Workspace management**: Numbers 1-9 for workspaces, 0 for workspace 10

### Remote Sync Configuration
- **Target host**: 100.119.167.52 (jparsell user)
- **Sync method**: rsync with exclusions for .git, .DS_Store, and *.swp files
- **Linux-only sync**: Pull operations exclude mac/ directory

### Testing and Validation
- **Tmux plugins**: Test with prefix + I (install), prefix + U (update)
- **Neovim plugins**: Launch nvim to trigger lazy loading
- **Shell config**: Source ~/.zshrc after changes
- **Window managers**: 
  - Aerospace: `aerospace reload-config`
  - Hyprland: `hyprctl reload`

When modifying configurations, always test on both platforms if making cross-platform changes, and ensure stow operations work correctly by testing symlink creation.

---
> Source: [jakep36/dotfiles](https://github.com/jakep36/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
