## nix-dots

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
# Apply system configuration (current host)
sudo nh os switch

# Apply for a specific host
sudo nixos-rebuild switch --flake .#bastion
sudo nixos-rebuild switch --flake .#forge

# Dry-run (preview changes without applying)
sudo nixos-rebuild dry-activate --flake .#<hostname>

# Format all Nix files
nix fmt

# Lint (run manually; CI uses statix + deadnix + alejandra)
nix run nixpkgs#statix -- check .
nix run nixpkgs#deadnix -- .

# Edit an agenix secret (requires YubiKey)
nix run .#edit-secret -- secrets/<name>.age

# Rekey all secrets (after adding a new host or key)
nix run .#rekey

```

## Architecture

### Flake Structure
- `flake.nix`: Defines inputs (nixpkgs `nixos-25.11`, `nixpkgs-unstable`, `home-manager`, `disko`, `agenix`, `lanzaboote`, `hyprpanel`, `neovim-nightly-overlay`) and all system outputs via a `mkHost` helper function.
- All systems are built with `mkHost`, which wires together a NixOS host module, a home-manager module, disk config, and special args (unstable pkgs, hyprpanel, vscode extensions).

### Hosts
- **`bastion`** — personal laptop, user `gast`, Intel GPU, single HiDPI monitor
- **`forge`** — work laptop, user `serobja`, Intel + NVIDIA GPU, 3-monitor setup; has additional work modules (VPN, certs, DNS, Docker registries)

### Module Layout
- `modules/core/` — users, boot (lanzaboote), disko, nix settings, nh cleanup
- `modules/desktop/` — Hyprland, SDDM, audio (PipeWire), Bluetooth, power management
- `modules/dev/` — Docker, Node, Python, PHP, editors
- `modules/hardware/` — Intel, NVIDIA, Thunderbolt dock
- `modules/security/` — YubiKey (PIV + FIDO2), agenix
- `modules/work/` — VPN, internal CAs, DNS, private Docker registries, work apps
- `modules/compat/` — distrobox, nix-ld
- `home/common/` — shared home-manager config (zsh, neovim-nightly, VS Code, Hyprland, rofi, kitty, starship, etc.)
- `home/bastion.nix` / `home/forge.nix` — per-host home-manager entrypoints

### Package Strategy
- Stable `nixpkgs` (`nixos-25.11`) for system packages
- `nixpkgs-unstable` for: Hyprland, HyprPanel, neovim (nightly overlay), VS Code, claude-code, and other fast-moving tools
- `nix-vscode-extensions` for declarative VS Code extension management

### Secrets (agenix)
- All secrets are age-encrypted in `secrets/*.age`
- `secrets/secrets.nix` maps each secret to the public keys that can decrypt it (two YubiKey masters + per-host SSH keys)
- Secrets are available at runtime under `/run/agenix.d/<name>`
- Adding a new secret: add entry to `secrets/secrets.nix`, run `nix run .#edit-secret`, then `nix run .#rekey`

### Disk / Boot
- **Disko**: LUKS2-encrypted BTRFS with subvolumes (`@`, `@home`, `@nix`, `@log`, `@swap`)
- **Lanzaboote**: Secure Boot (keys enrolled post-install via `sbctl`)
- **Hibernation**: Resume offset must be calculated per machine and set in `hosts/<name>/default.nix`
- **FIDO2**: YubiKey LUKS unlock via `systemd-cryptenroll` (passphrase fallback kept)

### Theming
- GTK: Catppuccin Mocha Mauve (dark)
- Color generation: Matugen derives palette from wallpaper, applied to HyprPanel and other components
- Fonts: JetBrainsMono Nerd Font (mono), Noto (sans/serif), Fira Code

## Formatting & Linting
The formatter is `alejandra` (set as `flake.formatter`). Linting is done with `statix` (idiomatic Nix checks) and `deadnix` (dead code). Run `nix fmt` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/robertjarske)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/robertjarske)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
