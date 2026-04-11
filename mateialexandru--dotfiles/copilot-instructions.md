## dotfiles

> This file provides guidance to Gemini CLI when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini CLI when working with code in this repository.

## Overview

Cross-platform dotfiles repository managing Doom Emacs configuration for macOS, Windows, and Linux (Bluefin/Fedora). Uses symlinks to link `doom/` to `~/.config/doom`.

## Setup

### macOS / Linux

```bash
./install.sh               # Installs Homebrew, Gemini CLI, Doom Emacs, and tools
```

This script:
- Installs Homebrew (if missing)
- Installs `gemini-cli` via npm
- Symlinks `doom/` to `~/.config/doom`
- Installs Doom Emacs and dependencies (ripgrep, fd, fonts)
- Syncs Doom configuration

### Windows

```powershell
.\install.ps1              # Installs Doom Emacs and dependencies via winget
```

## Doom Configuration

- `doom/init.el` — Module declarations
- `doom/packages.el` — Package declarations (symon is disabled on macOS)
- `doom/config.el` — Shared configuration
- `doom/config-macos.el` — macOS-specific settings (Cmd as Meta, etc.)
- `doom/config-linux.el` — Linux-specific settings
- `doom/config-windows.el` — Windows-specific settings

## Tools

- `gemini-cli` — AI assistant (installed via npm)
- `gh` — GitHub CLI
- `fzf` — Fuzzy finder
- `zoxide` — Smarter `cd`
- `hack` — (Optional) Worktree management script (shell/hack.ps1), requires PowerShell.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mateialexandru)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mateialexandru)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
