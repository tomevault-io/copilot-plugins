## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal dotfiles repository managed by [chezmoi](https://www.chezmoi.io/). The repository uses chezmoi's template system to support multiple machines (personal, work1, work2) with machine-specific configurations.

## Key Architecture Patterns

### Chezmoi Source Directory Structure

- `.chezmoiroot` file specifies `home/` as the source directory
- All dotfiles are located in `home/` directory, not at the repository root
- Template files use `.tmpl` extension (e.g., `dot_gitconfig.tmpl`)
- Private files use `private_` prefix (e.g., `private_dot_config/`)
- Dot prefix `dot_` converts to `.` in target location (e.g., `dot_zshrc` → `~/.zshrc`)

### Machine ID System

The repository uses a `machineId` variable for machine-specific configurations:

- `personal` - Personal machine with full dev tooling (nvm, pnpm, mise, direnv)
- `work1` - Work machine #1 with specific Google Cloud SDK path
- `work2` - Work machine #2 with different Google Cloud SDK path

Machine ID is configured in:
- `home/.chezmoi.toml.tmpl` - Prompts for machineId on first init
- `home/dot_gitconfig.tmpl` - Different git user/email per machine
- `home/private_dot_config/zsh/rc/local_env.sh.tmpl` - Machine-specific environment variables

### Automated Setup Scripts

Scripts in `home/.chezmoiscripts/` run automatically during chezmoi init:

1. `run_once_before_install-homebrew.sh.tmpl` - Installs Homebrew if missing
2. `run_once_after_install-packages.sh.tmpl` - Installs packages from Brewfile
3. `run_once_after_setup-nvim.sh.tmpl` - Sets up Neovim (hybrid theme, dein.vim, python support)

### Claude Code Configuration (`dot_claude/`)

The `dot_claude/` directory contains Claude Code settings and is mapped to `~/.claude/`:

```
home/dot_claude/
├── settings.json.tmpl     # Master config (permissions, MCP servers, plugins)
├── CLAUDE.md              # Project-level guidelines
├── agents/                # Specialized AI agents
├── skills/                # Reusable workflow skills
├── commands/              # Workflow entry point commands
└── rules/                 # Additional rule files
```

## Common Commands

### Chezmoi Operations

```bash
# Edit a dotfile
chezmoi edit ~/.zshrc

# Preview changes
chezmoi diff

# Apply changes to home directory
chezmoi apply

# Update from remote and apply
chezmoi update

# List managed files
chezmoi managed
```

### Package Management with Brewfile

```bash
# Install packages from Brewfile
brew bundle --file=~/dotfiles/home/Brewfile

# Update Brewfile with currently installed packages
brew bundle dump --file=~/dotfiles/home/Brewfile --force

# Check if all Brewfile packages are installed
brew bundle check --file=~/dotfiles/home/Brewfile

# Remove packages not in Brewfile (dry run)
brew bundle cleanup --file=~/dotfiles/home/Brewfile --dry-run
```

### Git Workflow for Dotfiles

The working directory is `~/dotfiles` (not `~/.local/share/chezmoi`):

```bash
cd ~/dotfiles
# Make changes to files in home/
git add -A
git commit -m "Update configuration"
git push
```

## Important Notes

- The actual chezmoi source directory is `~/dotfiles/home/`, not `~/dotfiles/`
- When adding new dotfiles, place them in `home/` directory with proper naming (e.g., `dot_filename`)
- Template files can use `{{ .machineId }}` to conditionally include machine-specific configurations
- Files listed in `home/.chezmoiignore` are not managed by chezmoi
- On new machine setup: `chezmoi init --apply mikanfactory/dotfiles`

---
> Source: [mikanfactory/dotfiles](https://github.com/mikanfactory/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
