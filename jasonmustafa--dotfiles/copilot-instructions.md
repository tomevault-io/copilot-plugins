## dotfiles

> This file provides guidance to AI coding agents when working with code in this

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this
repository.

## Project Overview

Personal dotfiles managed by chezmoi. Source directory is `home/` (set via
`.chezmoiroot`).

## Commands

```bash
chezmoi apply              # Apply all changes to home directory
chezmoi apply -v           # Apply with verbose output
chezmoi apply <target>     # Apply changes to single file (use ~ path)
chezmoi diff               # Preview changes before applying
chezmoi edit <file>        # Edit a managed file
chezmoi add <file>         # Add new file to chezmoi management
```

## Architecture

### Directory Layout

- `home/` - Chezmoi source (maps to `~`)
  - `dot_*` → `~/.{name}` (e.g., `dot_zshrc` → `~/.zshrc`)
  - `dot_config/` → `~/.config/`
  - `private_*` - Files with restricted permissions
  - `run_onchange_*` - Scripts that run when source changes
- `arch/` - Arch Linux package lists (not deployed by chezmoi)

### Templating

Files ending in `.tmpl` use Go templates with chezmoi data:

```go
{{ if eq .chezmoi.os "darwin" -}}
# macOS specific
{{- else -}}
# Linux specific
{{- end }}
```

### Key Configs

- **kitty**: `home/dot_config/kitty/kitty.conf.tmpl` - OS-conditional font
  size/effects
- **tmux**: `home/dot_config/tmux/tmux.conf.tmpl` - See subdirectory `AGENTS.md`
  for status bar architecture
- **nvim**: `home/dot_config/external_nvim/` - Git submodule (kickstart.nvim
  fork)

### Design Patterns

- Catppuccin Mocha theme throughout all configs
- Nerd Font glyphs embedded directly with trailing space for proportional width

## Nerd Font Glyphs

When editing lines containing Nerd Font icon glyphs:

- Only modify necessary characters around the glyph, never rewrite it
- Use `replace_all` on safe substrings when bulk renaming
- If glyph itself needs changing, use placeholder `X` with comment:
  `"X "  # f120 - terminal`

## Commit Message Rules

- Never include Arch package add/remove changes from `arch/*pkglist.txt` in
  commit message summaries or bodies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonmustafa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jasonmustafa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
