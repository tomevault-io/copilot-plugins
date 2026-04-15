## nix-files

> This repository contains a comprehensive [Nix](https://nixos.org/) configuration for managing macOS systems. It leverages [Nix Flakes](https://nixos.wiki/wiki/Flakes) for reproducible builds, [nix-darwin](https://github.com/LnL7/nix-darwin) to manage system-level settings, and [Home Manager](https://github.com/nix-community/home-manager) to manage user-level configurations and packages.

# Project Overview

This repository contains a comprehensive [Nix](https://nixos.org/) configuration for managing macOS systems. It leverages [Nix Flakes](https://nixos.wiki/wiki/Flakes) for reproducible builds, [nix-darwin](https://github.com/LnL7/nix-darwin) to manage system-level settings, and [Home Manager](https://github.com/nix-community/home-manager) to manage user-level configurations and packages.

The configuration is structured to support multiple machines with both `aarch64` and `x86_64` architectures, sharing a common base of settings and packages while allowing for machine-specific customizations.

## Key Technologies

*   **[Nix](https://nixos.org/):** A powerful package manager and system configuration tool that enables declarative and reproducible environments.
*   **[Nix Flakes](https://nixos.wiki/wiki/Flakes):** A new, improved way to manage Nix expressions and their dependencies, ensuring that every build is hermetic and reproducible.
*   **[nix-darwin](https://github.com/LnL7/nix-darwin):** A project that allows you to manage your macOS configuration using Nix, similar to how NixOS manages Linux systems.
*   **[Home Manager](https://github.com/nix-community/home-manager):** A tool to manage a user's home directory and dotfiles declaratively with Nix.

## Building and Running

To apply the configuration to a macOS system, use the following command:

```sh
darwin-rebuild switch --flake .
```

This command will build the system configuration based on the `flake.nix` file and apply it to the current machine. The specific configuration applied depends on the machine's hostname, which is used to select the corresponding `darwinConfiguration` from the `flake.nix`.

## Development Conventions

The project follows standard Nix conventions.

*   **Modularity:** The configuration is broken down into modules located in the `modules/` directory. The `shared/` subdirectory contains configurations that are common across all machines.
*   **Declarative Configuration:** All system and user configurations are defined declaratively in `.nix` files. This includes system settings, installed packages, and application configurations.
*   **Package Management:** Both system-level packages (via `environment.systemPackages`) and user-level packages (via `home.packages`) are managed by Nix. Homebrew is also used for a few specific packages that are not available or do not work well in Nixpkgs.
*   **Development Environment:** A development shell is defined in `flake.nix` using `devShells.default`. This shell provides common tools for working with the configuration itself, such as `nixpkgs-fmt` for formatting Nix code. You can enter this shell by running `nix develop`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/regadas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
