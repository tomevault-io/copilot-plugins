## nixos-config

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

System rebuild (run from the NixOS machine, uses `nh` nix-helper wrapper):
```bash
nh os switch          # rebuild and switch (fish abbr: nrs)
nh os test            # rebuild and test without boot entry (fish abbr: nrt)
nh os boot            # rebuild, activate on next boot (fish abbr: nrb)
```

Flake management:
```bash
nix flake update      # update all inputs (fish abbr: nfu)
nix flake check       # validate flake
```

There is no test suite or linter — validation is done by building (`nh os switch`).

## Architecture

**Single-flake, single-host NixOS configuration** for machine `lightspeed` (x86_64-linux).

### Flake inputs
- `nixpkgs` (unstable), `home-manager`, `stylix` (base16 theming), `nixvim` (Neovim config), `sops-nix` (secrets)

### Module layers

```
flake.nix
  └─ hosts/lightspeed/
       ├─ configuration.nix   # System entry point — imports all nixos/ modules + themes/
       ├─ home.nix             # Home Manager entry point — imports all home/ modules
       ├─ variables.nix        # Machine-specific values (user, monitors, paths, theme)
       └─ hardware-configuration.nix  # Generated, machine-specific
```

- **`nixos/`** — System-level modules (one file per concern: audio, bluetooth, boot, btrfs, docker, flatpak, gpu, greetd, locale, networking, nix, printing, sysctl, users)
- **`home/programs/`** — Per-application Home Manager configs (browser, editor, fish, ghostty, git, neovim, yazi, zellij)
- **`home/system/`** — Desktop environment modules (hyprland, waybar, fuzzel, mako, hypridle, hyprlock, swww)
- **`home/packages.nix`** — All user-level packages in one file
- **`home/scripts/`** — Helper bash scripts (power-menu, theme-toggle, screen-record)
- **`themes/default.nix`** — Stylix configuration (colors, fonts, opacity, cursor)

### Key patterns

**Variables pattern:** Each host has a `variables.nix` exporting machine-specific values. The flake imports these and passes them as `vars` via `specialArgs` (system modules) and `extraSpecialArgs` (Home Manager). Modules receive `vars` in their function arguments: `{ config, pkgs, vars, ... }:`. Do NOT use `let vars = import ...;` — that pattern is deprecated.

**`inputs` propagation:** The flake passes `inputs` via `specialArgs` (system modules) and `extraSpecialArgs` (Home Manager) so any module can access flake inputs like `nixvim`.

**Stylix theming:** Global base16 Solarized Dark theme applied via Stylix. Most apps inherit colors automatically; some (hyprlock, waybar) reference Solarized hex values directly.

**Home Manager integration:** Configured as a NixOS module (`useGlobalPkgs = true`, `useUserPackages = true`), not standalone.

**Filesystem:** btrfs on LUKS with subvolumes (@root, @home, @nix, @devel, @log, @snapshots, @swap). Automated snapshots via btrbk (hourly, retention 48h/14d/4w/3m). Monthly scrub. `/devel` is a shared workspace (group `devel`, setgid).

## Adding a new host

1. Create `hosts/<hostname>/` with `configuration.nix`, `home.nix`, `variables.nix`, and generated `hardware-configuration.nix`
2. Add a new `nixosConfigurations.<hostname>` entry in `flake.nix`

## Adding a new module

- System service → create `nixos/<name>.nix`, import it in `hosts/lightspeed/configuration.nix`
- User program → create `home/programs/<name>.nix`, import it in `hosts/lightspeed/home.nix`
- Desktop component → create `home/system/<name>.nix`, import it in `hosts/lightspeed/home.nix`

## CHANGEME markers

Search for `CHANGEME` to find values that must be customized per-installation (email, git username, monitor names, wallpaper hash, hwmon path).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kiperz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
