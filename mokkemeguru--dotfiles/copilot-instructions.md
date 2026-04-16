## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal dotfiles repository that manages configuration files for various development tools and environments across macOS and Arch Linux systems. The repository uses symlinks to deploy configurations and provides automated installation scripts.

## Key Architecture

### Installation System
- **Main installer**: `install.sh` - Entry point for setting up dotfiles
  - Supports OS detection (`-o` flag for `osx` or `arch`)
  - Option to load bash configurations (`-l` flag)
- **Installation pipeline**:
  1. `etc/init/{os}/init.sh` - OS-specific initialization
  2. `dotfilesDepends.sh` - Installs dependencies (fonts, package managers, tools)
  3. `dotfilesLink.sh` - Creates symlinks from dotfiles to home directory
  4. `dotfilesAfter.sh` - Post-installation tasks (plugin installation, directory creation)

### Configuration Structure
- **Shell configurations**:
  - `bash/.bashrc` and `bash/.bash_profile` - Bash configurations with aliases and environment setup
  - `zsh/.zshrc` and `zsh/.zshenv` - Zsh configurations with Sheldon and Starship integration
- **Editor configurations**:
  - `doom.d/` - Doom Emacs configuration (init.el, packages.el, config.el)
  - `vim/` - Vim configurations with Vundle plugin management
- **Tool configurations**:
  - `sheldon/plugins.toml` - Zsh plugin manager configuration
  - `starship/starship.toml` - Cross-shell prompt configuration
  - `textlint/.textlintrc.json` - Text linting configuration

## Common Development Commands

### Installation and Setup
```bash
# Install dotfiles for macOS
./install.sh -o osx -l

# Install dotfiles for Arch Linux
./install.sh -o arch -l

# After installation, change default shell
chsh -s $(which zsh)
```

### Doom Emacs Management
```bash
# Sync Doom Emacs packages after modifying packages.el
doom sync

# Upgrade Doom Emacs
doom upgrade

# Reload Doom configuration
doom reload
```

### Dependency Updates
```bash
# Update Sheldon plugins
sheldon lock --update

# Install Vim plugins
vim +PluginInstall +qall

# Update Rust tools (sheldon, starship)
cargo install sheldon --locked
cargo install starship --locked
```

## Environment Management

The repository manages multiple programming environments:
- **Python**: Uses pyenv for version management
- **Ruby**: Uses rbenv for version management  
- **Go**: Uses goenv for version management
- **Node.js**: Uses Volta for version management
- **Rust**: Uses rustup with cargo

## Key Dependencies

Essential tools installed by `dotfilesDepends.sh`:
- HackGen Nerd Font (custom installation script in `fonts/hackgen/`)
- Babashka (Clojure scripting)
- clj-kondo (Clojure linter)
- Language version managers (pyenv, rbenv, goenv, volta)
- Shell tools (sheldon, starship)
- Editor tools (Doom Emacs packages, Vim Vundle)

## Important Notes

- The repository supports both macOS and Arch Linux with OS-specific configurations
- Symlinks are created from the dotfiles directory to the home directory - modifications should be made in the dotfiles repository
- Shell configurations source each other: `.zshrc` sources `.bashrc` for shared aliases
- The repository includes extensive Doom Emacs customizations with LSP support for multiple languages
- Git configurations and aliases are defined in bash/zsh configs rather than separate git files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MokkeMeguru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
