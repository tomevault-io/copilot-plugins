## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- `make help` - List all available make commands with descriptions
- `make plugins` - Install all plugin managers using get_plugin_managers.sh
- `make nvim` - Build NeoVim Docker image (ghcr.io/hsteinshiromoto/dotfiles.linux/dotfiles.linux:nvim)
- `make changelog` - Update CHANGELOG.md using git-cliff
- `make tree` - Print directory tree structure (excludes .git and cache dirs)
- `make clean` - Remove log files
- `docker-compose up nvim` - Run NeoVim in Docker container on port 6666

## Repository Architecture

### Structure Philosophy
This dotfiles repository mirrors the `$HOME` directory structure exactly. Files should be placed in their target locations within the repo, then symlinked using `stow . --adopt`.

### Platform Organization
- **macos/**: macOS-specific configurations (Library, Application Support, etc.)
- **Root level**: Cross-platform dotfiles that go directly in `$HOME`
- **bin/**: Utility scripts (aliases, plugin managers)
- **utils/prompts/**: AI prompts and templates

### Key Components
- **Neovim**: Complex Lua configuration in `.config/nvim/lua/` with modular plugin structure
- **Espanso**: Text expansion configs in `macos/Library/Application Support/espanso/`
- **Docker**: NeoVim development environment with Ubuntu base (nvim.Dockerfile)
- **Stow**: Primary dotfile management tool for symlinking configurations

### Development Workflow
1. **New dotfiles**: Create file in repo location → `stow . --adopt` → `git add`
2. **Testing**: Use Docker environment for Linux testing
3. **Plugin management**: Run `make plugins` to install all managers

## Code Style Guidelines
- **Shell/Bash**: 4-space indent, lowercase_with_underscore variables
- **YAML**: 2-space indent, list items with single dash and space  
- **Lua (Neovim)**: Follow existing plugin structure patterns
- **Git commits**: Descriptive, feature/fix prefixed messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hsteinshiromoto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
