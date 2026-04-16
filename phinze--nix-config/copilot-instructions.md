## nix-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal Nix configuration repository that manages multiple machines using NixOS and nix-darwin. The owner primarily works in a tmux/neovim environment on NixOS VMs accessed via SSH, with macOS laptops as access points.

## Architecture

### Flake Structure
- `flake.nix` - Main entry point defining nixosConfigurations and darwinConfigurations
- `nixos/` - NixOS system configurations (foxtrotbase, xiezhi, simurgh, homegate)
- `nix-darwin/` - macOS system configurations (phinze-mrn-mbp)
- `home-manager/` - User environment configs shared across all machines
- `modules/` - Reusable modules for different platform types
- `overlays/` - Custom package overlays and modifications
- `pkgs/` - Custom packages

### Host Types
- **NixOS VMs**: Development environments with Docker, SSH, Tailscale
- **macOS laptops**: Managed by nix-darwin with Homebrew for GUI apps

### Key Technologies
- Nix flakes with inputs following nixpkgs stable and unstable
- Home Manager for user environment management
- Custom nixvim configuration (external flake)
- nix-index-database for command-not-found functionality
- Tailscale for networking between machines

## Common Commands

### Building and Switching Configurations

Using `nh` (nix-helper) for more convenient commands:

For NixOS systems:
```bash
# Build and switch on the current machine
nh os switch .

# Build and switch a specific host
nh os switch . -- --flake .#foxtrotbase
```

For macOS systems:
```bash
# Build and switch darwin configuration
nh darwin switch .

# Build and switch specific host
nh darwin switch . -- --flake .#phinze-mrn-mbp
```

Legacy commands (without nh):
```bash
# NixOS
sudo nixos-rebuild switch --flake .
sudo nixos-rebuild switch --flake .#foxtrotbase

# macOS
darwin-rebuild switch --flake .
darwin-rebuild switch --flake .#phinze-mrn-mbp
```

### Maintenance Commands

```bash
# Update flake inputs
nix flake update

# Update specific input
nix flake lock --update-input nixvim-config

# Format nix files
nix fmt

# Check flake
nix flake check
```

### Development Workflow

The owner uses a ghq-based directory structure under `~/src` for organizing git repositories, accessible through fish shell functions and tmux session management.

### Updating Package Hashes

When updating packages in `pkgs/`, use this command to get the correct SRI hash format:
```bash
nix-prefetch-url <url> | xargs -I {} nix hash to-sri --type sha256 {}
```

This converts the base32 hash from `nix-prefetch-url` to the SRI format (`sha256-...=`) that Nix derivations expect.

## Clipboard Access

Use `pbcopy` to copy text to the user's system clipboard. Always use a heredoc — piping from `echo` or `printf` results in 0 bytes reaching pbcopy in this environment:
```bash
cat <<'EOF' | pbcopy
Text to copy here.
Quotes, apostrophes, and special characters all work.
EOF
```
For copying file contents: `cat file.txt | pbcopy`

This works on both macOS (native) and NixOS (via OSC 52 through SSH/tmux).

## Claude Code Configuration

This repo is the source of truth for Claude Code configuration. Skills, CLAUDE.md files, and settings are managed declaratively via home-manager in `home-manager/phinze/claude-code.nix` and deployed as symlinks.

- **Global CLAUDE.md** → `home-manager/phinze/claude-global.md`
- **Skills** → `home-manager/phinze/claude-skills/*.md`
- **Settings, plugins, wiring** → `home-manager/phinze/claude-code.nix`

## Important Notes

- The repository uses Determinate Nix on macOS (nix.enable = false in darwin configs)
- SSH keys are managed in system configurations for inter-machine access
- Home-manager configurations are platform-aware using conditional logic
- All machines share the same user (phinze) with consistent shell environment

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phinze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
