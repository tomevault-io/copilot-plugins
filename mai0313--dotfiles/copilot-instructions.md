## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Chezmoi-managed dotfiles repo (`mai0313/dotfiles`). Deploys shell configs, IDE settings, fonts, and setup scripts to `$HOME` across personal machines, Google Cloudtop (work), and GitHub Codespaces.

## Common Commands

```bash
chezmoi diff          # Preview changes before applying
chezmoi apply         # Apply source state to $HOME
chezmoi re-add        # Sync local edits back to source directory
chezmoi add ~/.file   # Start managing a new file
chezmoi cd            # cd into this source directory
```

After editing templates, validate with `chezmoi execute-template < file.tmpl` or `chezmoi diff` to verify output.

## Architecture

### Chezmoi Naming Conventions

- `dot_*` -> files with leading `.` (e.g., `dot_zshrc.tmpl` -> `~/.zshrc`)
- `executable_*` -> deployed with execute permission (e.g., `executable_setup.sh` -> `~/setup.sh`)
- `empty_*` -> creates empty files (e.g., `empty_dot_zshenv` -> `~/.zshenv`)
- `.tmpl` suffix -> processed as Go templates with chezmoi data

### Environment Detection (`.chezmoi.toml.tmpl`)

Template variables set based on hostname/env:

| Variable | Condition | Use |
|---|---|---|
| `is_work` | hostname `*.c.googlers.com` | ADB keys, Gemini CLI, work aliases |
| `is_codespace` | env `CODESPACES=true` | Codespaces-specific config |

Used in templates as `{{- if .is_work }}...{{- end }}`.

### Key Files

- `.chezmoi.toml.tmpl` - chezmoi config, defines template variables
- `dot_zshrc.tmpl` / `dot_bashrc.tmpl` - shell configs (Zsh primary, Bash mirror)
- `dot_p10k.zsh` - Powerlevel10k prompt theme (lean style, NerdFont)
- `dot_claude/settings.json` - Claude Code settings
- `dot_gemini/settings.json` - Google Gemini settings
- `executable_setup.sh` - main setup script (oh-my-zsh, p10k, neovim, fonts)
- `executable_setup_adb.sh.tmpl` - work-only ADB vendor key setup (no-op on personal machines)
- `executable_cleanup.sh` - removes temp/cache dirs, preserves key config files
- `.chezmoiignore` - prevents `install.sh` and READMEs from being deployed to `$HOME`

### Shell Config Structure

Both `dot_zshrc.tmpl` and `dot_bashrc.tmpl` share the same pattern:
1. PATH extensions (Go, Rust, Cargo, Miniconda, Neovim)
2. NVM lazy loading
3. Common aliases (`cc='claude'`)
4. Conditional work block (Gemini alias, ADB_VENDOR_KEYS)
5. Editor selection (vim over SSH, nvim locally)

### Setup Scripts

Scripts are deployed as `~/setup.sh`, `~/setup_adb.sh`, `~/cleanup.sh`. They are **not** chezmoi hooks - they must be run manually after `chezmoi apply` on first-time setup. `setup.sh` handles both macOS (brew) and Linux (apt).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mai0313) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
