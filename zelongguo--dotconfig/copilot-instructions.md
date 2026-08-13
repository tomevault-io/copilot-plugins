## dotconfig

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a cross-platform (macOS + Linux) dotfiles repository. Configs are deployed via symlinks into `~/.config/` and `~/` by `install.sh`. The repo uses a single `main` branch — no merge conflicts by design.

## Repo architecture

```
shared/          → symlinked to ~/.config/<app>/ on both OSes
macos/           → symlinked to ~/.config/<app>/ on macOS only
linux/           → symlinked to ~/.config/<app>/ on Linux only
```

**Key layering pattern**: The shell entry point is `shared/zsh/zshrc` (symlinked to `~/.zshrc`). It sources Zim framework, then OS-specific zsh (`macos/zsh/macos.zsh` or `linux/zsh/linux.zsh`), then shared modules (`env.zsh`, `aliases.zsh`, `vim_keybindings.zsh`, `tmux.zsh`, `fzf_completion.zsh`). OS-specific zsh directories are NOT symlinked — they are sourced by the main zshrc from the repo path.

The zshrc detects the repo root dynamically by resolving its own realpath (it follows the symlink back to the repo), so the repo can be cloned anywhere.

## Deploy / update configs

```bash
./install.sh
```

This script:
1. Checks prerequisites (zsh, git, Homebrew)
2. Installs CLI tools from `Brewfile` (stable mode, `--no-upgrade`)
3. Detects OS and symlinks `shared/<app>/` → `~/.config/<app>/`
4. Symlinks OS-specific dirs from `macos/` or `linux/` → `~/.config/<app>/`
5. Symlinks `.zshrc`, `.zimrc`, `.tmux.conf` into `$HOME`

After deployment, reload with `source ~/.zshrc` or restart the terminal.

## Key config files

| File | Purpose |
|------|---------|
| `shared/zsh/zshrc` | Main zsh entry point, auto-detects OS and sources modules |
| `shared/zsh/zimrc` | Zim framework module list (prompt, completion, plugins) |
| `shared/zsh/aliases.zsh` | Shell aliases (`vim→nvim`, `g→lazygit`, `ra→yazi`, `t→tmux`) |
| `shared/zsh/env.zsh` | Environment variables |
| `shared/tmux/.tmux.conf` | Tmux config: prefix `Ctrl-a`, vi-mode copy, powerline, extrakto |
| `macos/aerospace/aerospace.toml` | macOS tiling WM config |
| `macos/kitty/kitty.conf` | macOS terminal config |
| `linux/i3/config` | Linux window manager config |
| `linux/polybar/config.ini` | Linux status bar config |
| `Brewfile` | Homebrew packages (neovim, tmux, yazi, lazygit, fzf, ripgrep, etc.) |

## Gitignore notes

- `.claude/` is gitignored — the repo's own `.claude/settings.json` won't be tracked
- `shared/nvim/` is gitignored (managed separately)
- `findings.md`, `progress.md`, `task_plan.md` are gitignored (session artifacts)
- Symlinks themselves are not committed; only the source config files in the repo are tracked

## Homebrew

All CLI tools are managed through Homebrew even on Linux. The `Brewfile` uses `--no-upgrade` to avoid unintended version bumps. `HOMEBREW_NO_AUTO_UPDATE=1` and `HOMEBREW_NO_INSTALL_CLEANUP=1` are set during install to speed things up.

## Shell framework

Uses **Zim** (not Oh-My-Zsh). Zim is installed separately to `~/.zim/`. The repo's `zimrc` defines which zim modules to load. Key modules: `environment`, `git`, `input`, `utility`, `asciiship` (prompt), `zsh-autosuggestions`, `fast-syntax-highlighting`, `fzf-tab`, `zsh-z`.

---
> Source: [ZelongGuo/dotconfig](https://github.com/ZelongGuo/dotconfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
