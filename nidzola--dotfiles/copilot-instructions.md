## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal dotfiles for Arch Linux and macOS workstations, managed with GNU Stow. All configs live in `stow/<package>/` and get symlinked to `$HOME` via stow.

## Key commands

```bash
# Full machine setup from scratch
./bootstrap.sh                  # auto-detects OS
./bootstrap.sh --packages       # install packages only
./bootstrap.sh --stow           # deploy configs only
./bootstrap.sh --dev            # install dev tools only
./bootstrap.sh --ssh            # deploy SSH config only
./bootstrap.sh --services       # enable services (Arch only)

# Stow a single package
stow -d stow -t $HOME <package>

# Unstow a single package
stow -d stow -t $HOME -D <package>

# Update tracked package lists (Arch only)
pacman -Qqen > pkglist.txt && pacman -Qqem > pkglist-aur.txt
```

## Architecture

- **`bootstrap.sh`** - Entry point for machine setup. Thin orchestrator that calls `install/*.sh` scripts.
- **`install/`** - Modular install scripts. Each is independently runnable:
  - `lib.sh` - Shared functions (logging, OS detection)
  - `packages.arch.sh` / `packages.mac.sh` - System packages
  - `services.arch.sh` - Systemd services, firewall, sshfs automount
  - `dev.sh` - Dev tools (Go, Rust, PostgreSQL, Redis, etc.)
  - `stow.sh` - Deploy all stow packages
  - `ssh.sh` - SSH config deployment and sshd hardening
- **`stow/`** - Config packages. Each subdirectory mirrors `$HOME` structure and gets symlinked by stow. Editing `~/.config/nvim/init.lua` edits the repo file directly.
- **`stow/bin/.local/bin/`** - Shell scripts deployed to `~/.local/bin` via stow.
- **`stow/ssh/.ssh/`** - SSH config with Include-based structure (`config.d/` for modular host configs).
- **`instructions/`** - Usage docs: `dotfiles-usage.md` (stow workflow) and `proxy.md` (Linux-to-Mac proxy chain for corporate network).

## Platform differences

- **Linux-only packages:** hypr, sway, waybar, mako, rofi, keyd, kitty, alacritty, aicommit2, zk
- **Shared packages:** zsh, tmux, git, nvim, nvim-custom, wezterm, lazygit, yazi, btop, bat, bin

## Adding a new config

1. Create `stow/<name>/` mirroring the home directory structure (e.g., `stow/foot/.config/foot/foot.ini`)
2. Add the package name to `PACKAGES` in `install/stow.sh`
3. Run `stow -d stow -t $HOME <name>`

## SSH multi-account setup

Two permanent SSH hosts for GitHub - no switching needed:
- `github.com` - personal key (`~/.ssh/id_ed25519`)
- `github.com-transcarent` - work key (`~/.ssh/id_transcarent`)

`.gitconfig` has scoped `url.insteadOf` so `go mod tidy` auto-routes transcarent private modules through the work key.

The `ggs` script in `stow/bin/.local/bin/` is kept as a backup for manual SSH switching if needed.

## Secrets

Machine-specific secrets go in `~/.zshrc.local` (sourced at end of `.zshrc`, never committed). Template: `stow/zsh/.zshrc.local.example`.

## Two nvim configs

- `stow/nvim/` - LazyVim-based config (default)
- `stow/nvim-custom/` - Custom config, launched via `NVIM_APPNAME=nvim-custom nvim` (aliased as `nvim-custom`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nidzola) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
