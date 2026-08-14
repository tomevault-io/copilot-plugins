## nix-dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Applying Changes

**NixOS system rebuild (includes Home Manager via NixOS module integration):**
```bash
sudo nixos-rebuild switch --flake .#<hostname>
# e.g.: sudo nixos-rebuild switch --flake .#zenith
```

**Standalone Home Manager switch (for non-NixOS or testing):**
```bash
home-manager switch --flake .#kqnade
# or for i3 environment:
home-manager switch --flake .#kqnade-i3
```

**Update flake inputs:**
```bash
nix flake update
```

**Check/evaluate flake without applying:**
```bash
nix flake check
```

## Architecture

This is a Nix flake-based dotfiles repo using **flake-parts** to manage both NixOS system configs and Home Manager user configs.

### Flake structure (`flake.nix`)
Uses `flake-parts` with modular outputs defined in:
- `hosts/default.nix` — `mkHost` helper defining `nixosConfigurations` with HM NixOS module integration
- `home/default.nix` — `mkHome` helper defining standalone `homeConfigurations`

Two separate nixpkgs inputs: `nixpkgs` (unstable) and `nixos` (25.11). An overlay provides `pkgs.unstable` in NixOS contexts so home packages always come from unstable.

### Flake outputs
- `nixosConfigurations`: `atraqutia` (minimal/VersaPro), `beltox` (i3wm/VersaPro), `zenith` (WSL)
- `homeConfigurations`: `kqnade` (common terminal), `kqnade-i3` (i3wm extended)

### Directory layout
- `hosts/` — Per-host NixOS configurations in subdirectories (`atraqutia/`, `beltox/`, `zenith/`). `default.nix` is the flake-parts module with `mkHost`.
- `hosts/hardconf/` — Shared hardware configurations.
- `modules/` — Shared NixOS modules: `packages.nix`, `system.nix`, `users.nix`. `default.nix` imports all three.
- `home/` — `default.nix` is the flake-parts module with `mkHome`.
- `home/kqnade/` — Home Manager config. `default.nix` is the common base; `i3.nix` extends it for i3wm.
- `home/kqnade/modules/` — Home Manager program modules: `git/`, `zsh/`, `nixvim/`, `direnv.nix`, `starship.nix`.

### NixVim structure (`home/kqnade/modules/nixvim/`)
Configured via [nixvim](https://github.com/nix-community/nixvim) home-manager module. Organized as:
- `plugins/` — Plugin configs grouped by category: `ui/`, `cmp/` (completion/LSP/lint/format), `lang/` (per-language), `tool/`
- `keymaps/` — Keybinding configs: `colemak.nix`, `edit.nix`, `window.nix`, `plugin.nix`
- `options.nix` — Neovim options

---
> Source: [kqnade/nix-dotfiles](https://github.com/kqnade/nix-dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
