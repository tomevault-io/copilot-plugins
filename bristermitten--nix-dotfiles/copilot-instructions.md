## nix-dotfiles

> This repository contains the Nix configuration for managing a macOS system (`nix-darwin`) and user environment (`home-manager`). It utilizes **Nix Flakes** and **nixos-unified** for a structured and modular configuration management approach.

# Project Context: NixConfig (macOS)

## Overview
This repository contains the Nix configuration for managing a macOS system (`nix-darwin`) and user environment (`home-manager`). It utilizes **Nix Flakes** and **nixos-unified** for a structured and modular configuration management approach.

## Key Technologies
- **Nix Flakes:** For reproducible dependency management.
- **nix-darwin:** macOS system configuration management.
- **home-manager:** User environment and dotfiles management.
- **nixos-unified:** Framework for structuring and autowiring Nix configurations.
- **flake-parts:** Component-based flake structure.
- **just:** Command runner for common tasks.

## Directory Structure
- **`flake.nix`**: The entry point of the configuration.
- **`configurations/`**:
    - **`darwin/`**: Host-specific system configurations (e.g., `Alexs-MacBook-Pro-3.nix`).
    - **`home/`**: User-specific Home Manager configurations (e.g., `alex.nix`).
- **`modules/`**: Reusable Nix modules.
    - **`darwin/`**: Modules specific to macOS settings.
    - **`home/`**: Modules for Home Manager (apps, git, shell, etc.).
    - **`flake-parts/`**: Flake-level configuration (e.g., `treefmt`, autowiring).
- **`justfile`**: Task runner definitions.

## Building and Running

The project uses `just` as a task runner. Ensure `just` is installed or use `nix run` directly.

### Common Commands
- **Apply Configuration:**
  ```bash
  just run
  # OR
  nix run
  ```
  This builds and activates the current darwin configuration.

- **Update Inputs:**
  ```bash
  just update
  # OR
  nix flake update
  ```

- **Format Code:**
  ```bash
  just lint
  # OR
  nix fmt
  ```
  Uses `treefmt` (configured with `nixpkgs-fmt`).

- **Check Flake:**
  ```bash
  just check
  ```

## Development Conventions
- **Autowiring:** The project relies on `nixos-unified`'s autowiring. Modules placed in standard directories (`modules/darwin`, `modules/home`) are automatically discoverable.
- **Host Configuration:** New hosts should be added to `configurations/darwin/`.
- **User Configuration:** User settings are managed in `configurations/home/` and linked via the host configuration.
- **Formatting:** All Nix files should be formatted using `nix fmt` before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bristermitten)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bristermitten)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
