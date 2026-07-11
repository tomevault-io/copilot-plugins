## dotfiles

> Guidelines for AI agents working in this Nix-based dotfiles repository.

# AGENTS.md - Dotfiles Repository Guide

Guidelines for AI agents working in this Nix-based dotfiles repository.

## Build/Lint/Test Commands

### Primary Commands

```bash
# Enter development shell (includes formatters, linters, clan-cli)
nix develop

# Format all files (Nix, JSON, YAML, Shell, Lua)
nix fmt

# Check flake for errors
nix flake check

# Update all flake inputs
nix flake update

# Update specific input
nix flake update nixpkgs
```

### System Build Commands

```bash
# NixOS system rebuild
sudo nixos-rebuild switch --flake .#<hostname>

# Darwin (macOS) system rebuild
darwin-rebuild switch --flake .#<hostname>

# Home-manager standalone
home-manager switch --flake .#<username>

# Via clan-core (preferred for multi-machine)
clan machines update <hostname>
clan machines list
clan machines show <hostname>

# Clan vars (machine secrets)
clan vars generate <hostname>
clan vars upload <hostname>
clan vars list <hostname>
```

### Hostnames Reference

- `pp-ml1` - Darwin laptop (Apple Silicon)
- `pp-ll1` - NixOS laptop (MacBook w/ T2)
- `pp-ld1` - NixOS desktop
- `pp-nas1` - NixOS NAS (BeeLink MeMini)
- `pp-router1` - NixOS router
- `pp-rpi1` - NixOS Raspberry Pi
- `pp-wsl1` - WSL instance

## Architecture Overview

This is a **dendritic Nix flake** using:

- **flake-parts**: Composable flake architecture
- **import-tree**: Automatic module discovery (all `.nix` files in `/modules` auto-imported)
- **clan-core**: Infrastructure-as-code machine deployment
- **GNU Stow**: Dotfiles deployment (runs automatically as a home-manager activation script)

### Module Export Pattern

Every module exports to `flake.modules.<platform>.<name>`:

```nix
# modules/example/foo.nix
_: {
  flake.modules.homeManager.foo = { pkgs, ... }: { ... };
  flake.modules.nixos.foo = { ... }: { ... };
  flake.modules.darwin.foo = { ... }: { ... };
}
```

## Code Style Guidelines

### Nix Formatting

Enforced via treefmt-nix with:

- `nixfmt` - Nix files
- `deadnix` - Remove dead code
- `statix` - Linting/suggestions
- `shfmt` - Shell scripts
- `stylua` - Lua files
- `prettier` - JS/TS/MD
- `jsonfmt` - JSON
- `yamlfmt` - YAML

Run `nix fmt` before committing.

### Nix Code Conventions

```nix
# Function arguments: use destructuring with trailing comma
{ inputs, config, lib, pkgs, ... }:

# Let bindings for complex expressions
let
  overlays = import ../../overlays { inherit inputs; };
in
{ ... }

# Use `with` sparingly, prefer explicit references
home.packages = with pkgs; [ ripgrep fd ];

# Module imports: use `with` for readability in import lists
imports = with modules.darwin; [ base rust ];

# Attribute sets: align colons for readability in small sets
{ name = "foo"; value = 42; }

# Platform conditionals
++ lib.optionals pkgs.stdenv.hostPlatform.isDarwin [ ... ]
++ lib.optionals pkgs.stdenv.hostPlatform.isLinux [ ... ]
```

### Naming Conventions

- **Machines**: `{prefix}-{os}{type}{num}` or `{prefix}-{role}{num}` for dedicated-role machines
  - Prefix: `pp` for personal
  - OS codes: `l`=Linux, `m`=macOS, `wsl`=WSL
  - Type codes: `l`=laptop, `d`=desktop
  - Role names: `nas`, `rpi` for dedicated-purpose machines
  - Examples: `pp-ll1` (Linux Laptop 1), `pp-nas1` (NAS 1), `pp-rpi1` (Raspberry Pi 1)
- **Modules**: lowercase, kebab-case filenames (e.g., `cloudflare-dns.nix`, `cloudflare-tunnel.nix`)
- **Export names**: camelCase (e.g., `cloudflareDns`, `cloudflareTunnel`, `routerFirewall`)
- **Functions**: camelCase in lib (e.g., `mkHomeConfigurations`, `relativeToRoot`)
- **Custom options**: Use `my.*` namespace (e.g., `my.router.*`, `my.jellyfin.*`, `my.pperanich.desktop`)
- **Upstream-style modules**: Use `services.*` only for modules structured like upstream NixOS modules (currently only `kimaki`)

### Directory Structure

```
/modules/           # Auto-imported modules (dendritic pattern)
  flake-parts/      # Flake infrastructure (clan, nixpkgs, fmt, shell)
  desktop/          # Desktop apps (fonts, applications, sketchybar)
  router/           # NixOS router framework (firewall, DHCP, DNS, VLANs)
  services/         # Self-hosted services (immich, nextcloud, vaultwarden, etc.)
  shell/            # Shell environment (nvim, opencode, rust, tools)
  system/           # Core system configs (base, sops)
  users/            # User account modules (pperanich)
/machines/          # Host-specific configurations
/home-profiles/     # User environment compositions
/home/              # Raw dotfiles (symlinked via stow activation)
/docs/              # Guides (adding machines, troubleshooting)
/lib/               # Custom library functions (lib.my.*)
/overlays/          # Nixpkgs overlays
/pkgs/              # Custom package definitions
/sops/              # Encrypted secrets
/vars/              # Non-secret variables
```

### Error Handling

- Prefer `lib.mkIf` for conditional enabling
- Use `lib.optionals` for conditional list items
- Use `lib.mkDefault` for overridable defaults
- Never use `builtins.throw` unless truly unrecoverable

### Module Best Practices

1. Single responsibility per module
2. Export to correct platform (`nixos`, `darwin`, `homeManager`)
3. Provide sensible defaults
4. Use `lib.my.relativeToRoot` for path references
5. Minimize cross-module dependencies

## Special Files and Patterns

### Extended Library

Custom functions available as `lib.my.*`:

- `lib.my.relativeToRoot` - Path relative to flake root
- `lib.my.getHomeDirs` - Discover home profile directories
- `lib.my.mkHomeConfigurations` - Generate home-manager configs

### Overlays (`/overlays/default.nix`)

Applied globally to nixpkgs. Use for:

- Version pinning
- Patch application
- Package modifications

### Secrets (Hybrid Approach)

Two systems handle secrets with different strengths:

- **Clan vars** — Machine bootstrap secrets (age keypairs, SSH host keys, user passwords, wireguard keys). Generated and uploaded via clan CLI.
- **sops-nix** — Application secrets (service passwords, API tokens). Encrypted in `sops/secrets.yaml`, decrypted at activation time using the machine's age key. Modules reference secrets via `sops.secrets.<name>.path`.

```bash
# Clan vars
clan vars generate <hostname>       # Generate machine secrets
clan vars upload <hostname>         # Upload secrets to machine
clan vars list <hostname>           # List vars for machine

# sops-nix
sops sops/secrets.yaml              # Edit secrets (requires age key)
```

Never commit plaintext secrets.

## Pre-commit Hooks

Configured via git-hooks.nix:

- `nix-fmt`: Auto-format on commit

## Common Pitfalls

1. **Module not found**: Check export path matches `flake.modules.<platform>.<name>`
2. **Stow conflicts**: Check `/home` directory for file conflicts
3. **State version mismatch**: Keep `stateVersion` consistent
4. **Overlay order**: Overlays in `/overlays/default.nix` apply globally

## Testing Changes

```bash
# Quick syntax check
nix flake check

# Build without switching (dry-run)
nixos-rebuild build --flake .#<hostname>
darwin-rebuild build --flake .#<hostname>

# Show what would change
nvd diff /run/current-system result
```

## Commit Guidelines

- Use conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`
- Test locally before committing
- Run `nix fmt` before commit (enforced by pre-commit hook)

---
> Source: [pperanich/dotfiles](https://github.com/pperanich/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
