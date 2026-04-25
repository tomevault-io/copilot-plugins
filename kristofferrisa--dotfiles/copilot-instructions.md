## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal dotfiles repository for system configuration management. It uses GNU Stow for symlink management and contains configurations for shell, terminal emulator, window manager, and development tools. The repository also includes a custom Claude Code workspace setup and OpenCode agent configurations.

## Installation & Setup Commands

### Initial Installation

```bash
# Install dotfiles and dependencies
./install.sh
```

The `install.sh` script:

- Installs base dependencies: stow
- Clones dotfiles repository if not present
- Installs LazyVim starter configuration
- Uses GNU Stow to symlink configurations to `~/.config/`

### Managing Configurations with Stow

Configurations are organized by application in `.config/` and linked using GNU Stow:

```bash
# Link specific config directory
stow -t ~/.config .config/<app>/

# Remove symlinks
stow -D -t ~/.config .config/<app>/

# Restow (useful after updates)
stow -R -t ~/.config .config/<app>/
```

## Architecture & Structure

### Configuration Organization

```
.config/
├── opencode/          # OpenCode agent configurations
│   └── agent/         # Agent prompt files (companion, senior-developer, etc.)
├── tmux/              # Tmux configuration and scripts
├── zsh/               # Zsh shell configuration
├── ghostty/           # Ghostty terminal emulator config
└── yabai/             # Yabai window manager config

.claude/
├── agents/            # Claude Code agents (git-commit-summarizer, markdown-optimizer)
└── settings.json      # Claude Code settings

```

### Agent System Architecture

This repository defines two parallel agent systems:

**OpenCode Agents** (`.config/opencode/agent/`):

- Defined as markdown files with YAML frontmatter
- Include: companion, senior-developer, solution-architect, code-reviewer, test-engineer, technical-writer, devops-engineer, diagram-specialist, requirements-analyst
- Primary agent is `companion` (read-only, research-focused)
- Specialized agents have different tool permissions and capabilities

**Claude Code Agents** (`.claude/agents/`):

- git-commit-summarizer: Analyzes changes and creates comprehensive commits
- markdown-optimizer: Ensures markdown files follow standards with proper frontmatter

Agent configurations specify:

- Model selection and temperature
- Available tools (write, edit, bash, webfetch, read, etc.)
- Permission policies (bash command restrictions)
- Operational mode (primary/specialized)

### Shell Environment

**Zsh Configuration** (`.config/zsh/.zshrc`):

- Oh-My-Zsh with Powerlevel10k theme
- Git plugin enabled
- Key aliases:
  - `n` - Open neovim in current directory
  - `gaa` - Git add all
  - `gcm` - Git commit with message
  - `gpsh` - Git push
  - `gss` - Git status short
  - Docker shortcuts: `dtable`, `dstart`

**Tmux Configuration** (`.config/tmux/tmux.conf`):

- Prefix: `Ctrl+a` (instead of default Ctrl+b)
- Vim keybindings in copy mode
- Custom split shortcuts: `|` (vertical), `-` (horizontal)
- Pane navigation: `h/j/k/l`
- Pane resizing: `Ctrl+a` then `h/j/k/l` (repeatable)
- Mouse support enabled
- History limit: 10,000 lines

## Development Workflow

### Adding New Configurations

1. Place config files in `.config/<application>/`
2. Test by stowing: `stow -t ~/.config .config/<application>/`
3. Verify symlinks created correctly
4. Commit changes to repository

### Modifying Agents

**OpenCode agents**: Edit markdown files in `.config/opencode/agent/`

- Update YAML frontmatter for model, temperature, tools, permissions
- Modify agent personality and capabilities in markdown body

**Claude Code agents**: Edit markdown files in `.claude/agents/`

- Follow agent description format with usage examples
- Include tool specifications and operating principles

## Key Configuration Patterns

### Permission-Based Agent Design

Agents use granular permission controls:

```yaml
permission:
  bash:
    "rm *": deny
    "rm -*": deny
    "sudo *": ask
    "*": allow
```

### Frontmatter Standards

Markdown files follow consistent frontmatter patterns:

- Agent configs: description, mode, model, temperature, tools, permissions
- Output files (fabyt): tags array, url metadata
- Documentation: title, date, tags

### Stow-Based Linking

Configurations are NOT copied but symlinked:

- Allows editing configs in dotfiles repo directly
- Changes immediately affect active system
- Easy to version control and sync across machines

## Notes for Future Modifications

- The companion agent is read-only focused (write: false, edit: false)
- Other OpenCode agents have write capabilities for code implementation
- Tmux config sources from `~/.config/tmux/tmux.conf` (not default location)
- Zsh config sources tmux.zsh for tmux-specific shell integration
- Custom scripts in `bin/` should include dependency checks and help text
- When adding new tools, update `brew-install.sh` for macOS dependencies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KristofferRisa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
