## nix-configuration

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Personal NixOS + Home Manager flake for multiple hosts (`deadPc`, `deadConvertible`, `deadServer`, `deadWsl`, `deadPi`). Built on `flake-parts`. See `README.md` for the layout overview.

## Commands

Rebuild uses `nh` (nix-helper), aliased in `modules/home-manager/core/aliases.nix`:

- `nos` → `nh os switch .` — rebuild the current host's NixOS config
- `nhs` → `nh home switch .` — rebuild Home Manager for the current user
- `nosf` / `nhsf` — same but with `--update` (bumps the flake lock first)

Lower-level equivalents:
- `sudo nixos-rebuild switch --flake .#<host>`
- `home-manager switch --flake .#deadmade@<host>`
- For `nix-mineral` (hardened) hosts like `deadPc`, prefer `nixos-rebuild boot` + reboot so the prior generation stays bootable for rollback.

Checks & formatting:
- `nix flake check` — custom outputs (`nixosProfiles`, `homeManagerModules`, etc.) emit informational warnings; these are expected.
- `nix fmt` / `alejandra` — formatter. Run via `nix develop` which installs a pre-commit hook (alejandra, convco for conventional commits, trufflehog secret scan, merge-conflict & large-file checks). **Commit messages must be Conventional Commits** (convco enforces this).
- `nix develop` — dev shell with `sops`, `age`, `cachix`, `vulnix`, `lazygit`, `ripgrep`.

## Architecture

The flake wiring lives in `flake/modules/` (each imported by `flake.nix`):

- `constants.nix` — defines `vars` (username, git identity, browser, terminal, keyboard layout) and loads `hosts/hosts.nix`. These are injected as `_module.args` so every flake module can see them.
- `exports.nix` — imports the four registries below and exposes them **both** as flake outputs (`nixosModules`, `homeManagerModules`, `nixosProfiles`, `homeManagerProfiles`, `overlays`) **and** as `projectOutputs` for internal use.
- `hosts.nix` / `home.nix` — iterate `hostDefinitions` to build `nixosConfigurations` and `homeConfigurations`. Both pass `specialArgs`/`extraSpecialArgs` of `{ inputs, vars, systems, outputs = projectOutputs }`. So inside any module, **`outputs` is this repo's own registry** (not the standard flake-utils `outputs`).
- `per-system.nix` — formatter, dev shell, and git pre-commit hooks.

### The module → profile → host layering

Everything composes through plain NixOS/Home-Manager modules (attrsets); there are **no custom `mkOption` definitions** — composition is done purely by importing.

1. **Modules** (`modules/nixos/`, `modules/home-manager/`): leaf config grouped by domain. Each domain dir has a `default.nix` that re-exports its files as a named attrset (e.g. `modules/nixos/desktop/default.nix` exposes `base`, `packages`, `stylix`, `vpn`, ...). The top-level `modules/*/default.nix` aggregates domains into the registry.
2. **Profiles** (`profiles/nixos/`, `profiles/home-manager/`): bundles of modules. A profile imports modules by reference, e.g. `outputs.nixosModules.desktop.base`, and profiles import other profiles (`outputs.nixosProfiles.core`). `core.nix` uses `builtins.attrValues outputs.nixosModules.core` to import a whole domain at once.
3. **Hosts** (`hosts/<name>/config.nix`, `home.nix`): import one or more profiles + host-specific hardware/overrides. `hosts/hosts.nix` is the single source of truth mapping host name → `{ system, nixosModules, homeModule, enableHome, ... }`.

**To add a new module:** create the `.nix` file, register it in the domain's `default.nix`, then reference it from a profile or host. Registration is required — files are not auto-discovered.

**To add a host:** add an entry to `hosts/hosts.nix` and create `hosts/<name>/`.

### Overlays

`overlays/default.nix` defines:
- `unstable-packages` — makes `nixpkgs-unstable` available as `pkgs.unstable.*` (used for e.g. `pkgs.unstable.vlc`).
- `flake-inputs` — aliases each flake input's packages to `pkgs.inputs.<name>`.
- `modifications` — package patches/overrides.

Overlays + `allowUnfree` are wired in `modules/nixos/core/defaults.nix` and `modules/home-manager/core/nixConfig.nix`. Both `nixpkgs` (26.05) and `nixpkgs-unstable` are tracked; pull individual packages from unstable via `pkgs.unstable`.

### Secrets

`sops-nix` with `age`. Encrypted YAML in `secrets/`, rules in `.sops.yaml`. Each host that needs secrets imports a `sops.nix` (e.g. `hosts/deadPc/sops.nix`) pointing at its sops file; the age key is read from `~/.config/sops/age/keys.txt`.

---
> Source: [deadmade/nix-configuration](https://github.com/deadmade/nix-configuration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
