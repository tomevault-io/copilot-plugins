## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# dotfiles

This project is a collection of configuration files to be checked out in `$HOME/.dotfiles` with files either linked to a '.dotfile' in the parent home directory, or referenced by other dotfiles.
It also includes any setup files to allow customization of a new machine.

## Architecture and Setup

### Installation Pattern
This dotfiles repository uses a **mixed approach** combining symbolic links and include/source statements:

**Automated Installation:**
Run `./install.sh` to automatically create symlinks for supported dotfiles:
- Checks for conflicts before creating links
- Creates symlinks: `~/.vimrc`, `~/.vim`, `~/.fzf.bash`, `~/.gemrc`, `~/.grc`, `~/.rgconfig`
- Provides warnings for missing source files

**Manual Configuration Required:**
- Main `~/.gitconfig` includes `.dotfiles/gitconfig` via `[include] path = .dotfiles/gitconfig`
- Main `~/.bash_profile` sources dotfiles scripts conditionally

### Vim Plugin Management
- Uses Vim 8+ native package management (`vim/pack/plugins/start/`)
- Vim plugins managed as **git submodules**
- See `update_vim_plugins.sh` to initialize, checkout and update vim plugins

### Key Configuration Files
- `install.sh` - Automated installation script for creating symlinks
- `bash_profile` - Main shell configuration with tool integrations
- `vimrc` - Vim configuration with extensive plugin setup
- `tmux.conf` - tmux terminal multiplexer configuration
- `alacritty.yml` - Alacritty terminal emulator configuration
- `gitconfig` - Git configuration with aliases and settings

### Shell Enhancement Scripts
- `make.sh` - Makefile tab completion
- `go.sh` - Go development utilities and navigation
- `rbenv.sh` - Ruby version management integration
- `fzf.bash` - Fuzzy finder setup and key bindings

### Development Environment
- **Go development**: Custom functions for package navigation in `go.sh`
- **Ruby development**: optional rbenv configuration in `rbenv.sh`
- **Git workflow**: Aliases defined in `gitconfig`

## Important Notes
- Conditional loading pattern: scripts check for tool existence before sourcing
- Designed for macOS and linux environments
- Optimized for Go/Ruby development with vim and tmux

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivan3bx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ivan3bx)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
