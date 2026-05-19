## nix-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# NixOS rebuild (current host)
just rebuild

# Update flake inputs
just update

# Test/validate without switching
just test                     # Test NixOS config
just dry-run                  # Preview what would change
just check                    # Validate flake

# Secrets
just secrets-edit <file>      # Edit encrypted secret
just secrets-rekey            # Re-encrypt after key change
just host-key <hostname>      # Get host's age public key

# Maintenance
just gc                       # Garbage collect old generations
just deploy <host>            # Deploy to remote host via SSH

# Install (ERASES DISK)
just install <hostname> <target-host>  # nixos-anywhere install with pre-seeded host key
```

## Architecture

Single-host NixOS flake with home-manager as a NixOS module.

### Hosts

- `nixosConfigurations.hl-node01` - Headless Incus server (AMD Ryzen 6800H)

Host entry: `hosts/<hostname>/default.nix`
Common config: `hosts/common/`

### Modules

```
modules/
├── nixos/           # NixOS system modules
│   ├── core/        # boot, nix, packages
│   ├── networking/  # firewall, ssh, tailscale
│   ├── storage/     # ZFS
│   ├── secrets/     # SOPS-nix
│   └── desktop/     # GNOME, Hyprland, KDE (optional)
└── home-manager/    # Home-manager profiles
    ├── profiles/    # Composable (core, shell, cli, development, gnome, hyprland, kde)
    └── users/       # User-specific config
```

### Secrets (SOPS + age)

```
secrets/
├── hosts/common/    # Host secrets (user + host keys)
└── users/goksel/    # User secrets (user key only)
```

SOPS rules in `.sops.yaml`. SSH host key converts to age key for decryption.

The user's password is stored as a hashed secret at `users/goksel/hashedPassword`
in `secrets/hosts/common/secrets.yaml`, decrypted with `neededForUsers = true`
so it's available before user creation. There is no plaintext default password.

The fresh-install bootstrap pre-seeds `/etc/ssh/ssh_host_ed25519_key` via
`nixos-anywhere --extra-files` so sops-nix can decrypt on the very first
activation. See `just install` for the flow.

## Key Patterns

- `mkHost` helper in `flake.nix` creates the system config
- Home-manager profiles imported via `hosts/common/users.nix`
- Per-host home-manager additions in host's `default.nix`

---
> Source: [gokselk/nix-config](https://github.com/gokselk/nix-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
