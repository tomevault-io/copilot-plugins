## workspace-setup

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Repository Purpose

This repository contains automated setup scripts and configuration files for
quickly setting up development environments on new laptops or VMs across Linux
(Arch, Ubuntu, Fedora) and Windows platforms.

> **Read [LEARNINGS.md](LEARNINGS.md) before modifying install logic** — it
> records non-obvious gotchas (upstream renames, broken pinned URLs, package
> quirks). Add an entry whenever a fix turns out to be non-obvious.

## Repository Structure

The repository is organized by operating system:

- `linux/arch/` - Arch Linux setup scripts and package list
- `linux/ubuntu/` - Ubuntu setup scripts (`install-zsh-ubuntu.sh`,
  `config-shell-tools-ubuntu.sh`, `setup-tmux.sh`) and `pkglist.txt`
- `linux/fedora/` - Fedora setup scripts (`setup.sh` entry point →
  `install-zsh-fedora.sh`, `config-shell-tools-fedora.sh`), optional
  `install-infra-fedora.sh`, `check-dotfiles.sh` audit tool, and `pkglist.txt`
- `windows/` - Windows PowerShell setup script
- `config/shell/` - Shared shell configuration files (`.zshrc`,
  `starship.toml`, `.tmux.conf`)
- `config/backup/` + `config/systemd/` - Encrypted Obsidian vault backup
  (rclone crypt → Google Drive) script and systemd user units

## Script Architecture

### Linux Scripts Pattern

All Linux setup scripts follow a consistent architecture:

1. **Dependency Checking**: Use `command -v` to check if tools are installed
   before attempting installation
2. **Idempotent Installations**: Check for existing installations (e.g.,
   `[ ! -d "$HOME/.oh-my-zsh" ]`) to avoid redundant operations
3. **Configuration Path Resolution**: Use
   `SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"` to locate
   scripts, then reference shared config files via relative paths like
   `$SCRIPT_DIR/../../config/shell`
4. **Silent Installation**: Use appropriate flags (`--noconfirm` for pacman,
   `-y` for apt/dnf, `RUNZSH=no KEEPZSHRC=yes` for the oh-my-zsh installer) to
   avoid interactive prompts

### Key Installation Flow

**Arch Linux** (`linux/arch/install-zsh-arch.sh`):

- Installs base tools via pacman (zsh, git, unzip, zip, wget, base-devel)
- Installs yay (AUR helper) from source if missing
- Installs Starship prompt via yay
- Installs Oh My Zsh framework
- Clones zsh plugins (autosuggestions, syntax-highlighting) to
  `~/.oh-my-zsh/custom/plugins/`
- Downloads and installs Hack Nerd Font to `~/.local/share/fonts`
- Copies shared config files from `config/shell/` to home directory

**Ubuntu** (`linux/ubuntu/install-zsh-ubuntu.sh`):

- Similar flow but uses apt instead of pacman
- Uses curl for Starship installation instead of yay
- Uses wget for Oh My Zsh installation

**Fedora** (`linux/fedora/setup.sh`):

- Single entry point running `install-zsh-fedora.sh` (shell foundation) then
  `config-shell-tools-fedora.sh` (dev tools, apps, 1Password CLI, vault backup)
- `install-infra-fedora.sh` is optional/opt-in (kubectl, k9s, flux, helm,
  kustomize) and deliberately NOT run by `setup.sh`
- `check-dotfiles.sh` audits deployed dotfiles/tools against the repo and
  offers interactive fixes

**Additional Tools** (`config-shell-tools-{arch,ubuntu,fedora}.sh`):

- Installs packages from the distro's `pkglist.txt`
- Clones LazyVim starter config to `~/.config/nvim`
- Installs nvm and Node.js 24 (current LTS)
- Installs Copilot.vim to `~/.config/nvim/pack/github/start/copilot.vim`
- Installs the Obsidian CLI (`notesmd-cli`, formerly `obsidian-cli` — see
  LEARNINGS.md) via `go install`

### Windows Script Architecture

The PowerShell script (`windows/restore-windows-apps.ps1`) follows a different
pattern:

- Requires Administrator privileges
- Uses both winget and Chocolatey package managers, via
  `Install-WingetApp`/`Install-ChocoPackage` helpers that check
  `$LASTEXITCODE` (native commands don't throw PowerShell errors — try/catch
  around them is dead code)
- Enables Windows features via DISM (WSL, Virtual Machine Platform)
- Installs applications via winget (Chrome, VSCode, Obsidian, Steam, Telegram,
  Ubuntu)
- Installs applications via Chocolatey (Surfshark VPN, vcredist140, Everything,
  Starship)
- Checks installed Microsoft Store apps via `Get-AppxPackage` and installs
  missing ones

## Shell Configuration Details

### .zshrc Configuration

Located in `config/shell/.zshrc`:

- Enables extensive history management (10M entries) with deduplication
- Adds `~/.zsh_functions` to `fpath` for custom completions — this MUST stay
  before the `source $ZSH/oh-my-zsh.sh` line (compinit runs there; completions
  added to fpath afterwards never register)
- Plugins: git, sudo, history, encode64, copypath, kubectl,
  zsh-autosuggestions, zsh-syntax-highlighting
- Tool aliases: `cd=z` (zoxide), `cat=bat`, `lg=lazygit`, `vim=nvim`
- Custom functions: `y()` for yazi file manager with directory changing
- Integrations: starship prompt, direnv, fzf, zoxide (init kept last)

### Starship Configuration

Located in `config/shell/starship.toml`:

- Uses Nerd Font symbols for various language and tool indicators
- Kubernetes integration enabled
- Custom git branch styling (#ffcce1)

## Development Commands

### Testing Linux Scripts

```bash
# Arch Linux
cd linux/arch
./install-zsh-arch.sh
./config-shell-tools-arch.sh

# Ubuntu
cd linux/ubuntu
./install-zsh-ubuntu.sh
./config-shell-tools-ubuntu.sh
./setup-tmux.sh

# Fedora
cd linux/fedora
./setup.sh
```

### Testing Windows Script

```powershell
# Run as Administrator
cd windows
.\restore-windows-apps.ps1
```

### Checking Script Syntax

```bash
# Bash scripts
bash -n <script-name>.sh

# PowerShell scripts (on Linux with pwsh)
pwsh -File <script-name>.ps1 -WhatIf
```

## Important Implementation Notes

### When Modifying Scripts

1. **Path Resolution**: Always use `SCRIPT_DIR` pattern to locate config
   files - never use hardcoded paths
2. **Idempotency**: Scripts should be safe to run multiple times without
   breaking or duplicating installations
3. **Silent Mode**: Keep all installations non-interactive for automation
   purposes
4. **Error Handling**: Use conditional checks (`command -v`, `[ ! -d ]`,
   `[ ! -e ]`) before installations
5. **Font Installation**: Hack Nerd Font v3.4.0 is the standard - URLs should
   point to this specific version
6. **Plugin Paths**: Zsh plugins go to
   `${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/`

### Configuration File Management

The config files in `config/shell/` are shared across all Linux distributions.
When modifying:

- Test changes on both Arch and Ubuntu (and Fedora, the primary daily driver)
- Ensure tool-specific configurations (like zoxide, fzf) work when tools are
  installed
- Keep aliases minimal and focused on common developer tools

### Package Management

**Arch** (`linux/arch/pkglist.txt`):

- Contains comprehensive system setup including GNOME desktop packages
- Official-repo packages ONLY — pacman aborts the entire transaction on any
  unknown target, so AUR-only packages must never be listed here
- Includes development tools: neovim, lazygit, tmux, bat, fd, fzf, ripgrep,
  yazi, zoxide
- System tools: btrfs-progs, snapper, zram-generator
- ASUS-specific: asusctl, supergfxctl

**Ubuntu** (`linux/ubuntu/pkglist.txt`):

- Base build/dev tools (build-essential, cmake, golang-go) and CLI utilities;
  the heavier tools (neovim, lazygit, yazi, ...) are installed from upstream
  releases in `config-shell-tools-ubuntu.sh`

**Fedora** (`linux/fedora/pkglist.txt`):

- Dev/CLI tools only — Fedora KDE ships the base system and desktop, so the
  list omits anything preinstalled (curl is deliberately excluded, see
  LEARNINGS.md)

---
> Source: [purplespacecat/workspace-setup](https://github.com/purplespacecat/workspace-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
