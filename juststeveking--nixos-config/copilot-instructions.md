## nixos-config

> This file provides a comprehensive overview of this NixOS configuration repository to be used as instructional context for an AI assistant.

# GEMINI.md: Project Overview

This file provides a comprehensive overview of this NixOS configuration repository to be used as instructional context for an AI assistant.

## Project Overview

This is a complete NixOS configuration managed using [Nix Flakes](https://nixos.wiki/wiki/Flakes). It sets up a personalized desktop environment for a machine named "framework". The configuration is modular, separating system-level settings from user-specific dotfiles, which are managed by [home-manager](https://github.com/nix-community/home-manager).

The primary goal is to create a declarative, reproducible, and version-controlled system environment.

### Key Technologies & Features

*   **Operating System:** NixOS Unstable
*   **Configuration Management:** Nix Flakes
*   **User Environment:** Home Manager
*   **Window Manager:** Hyprland (Wayland compositor)
*   **System Services:**
    *   `pipewire` for audio.
    *   `podman` for OCI containers.
    *   `ollama` for running local large language models.
    *   `greetd` with `tuigreet` for a TTY-based login manager.
*   **Creative Tools:**
    *   `kdenlive` for video editing.
*   **Structure:**
    *   System configuration is defined in `configuration.nix`.
    *   User-specific setup is in `home.nix`.
    *   Application configurations are modularized and imported from the `programs/` directory.

## Building and Running the System

The `Makefile` provides a set of convenient commands for managing the NixOS system.

*   **Apply the configuration:**
    ```bash
    make switch
    ```
    This command runs `sudo nixos-rebuild switch --flake .#$(hostname)` to build and activate the new configuration immediately.

*   **Test the configuration:**
    ```bash
    make test
    ```
    This builds the configuration and creates a temporary GRUB entry to test it without making it the default.

*   **Update flake inputs:**
    ```bash
    make update
    ```
    This command updates all the dependencies (Nixpkgs, home-manager, etc.) in the `flake.lock` file.

*   **Clean up old generations:**
    ```bash
    make gc
    ```
    This removes old, unused Nix store paths to free up disk space.

*   **Check the configuration:**
    ```bash
    make check
    ```
    This runs `nix flake check` to ensure the flake is syntactically correct and can be evaluated.

## Development Conventions

This repository follows a clear, modular structure to manage complexity.

*   **Central Flake:** `flake.nix` is the entry point. It defines all inputs and orchestrates the assembly of the final NixOS and home-manager configurations.
*   **System vs. User:**
    *   `configuration.nix`: Contains system-wide settings, hardware configuration, and services that run for the entire machine.
    *   `home.nix`: Is the main file for the `home-manager` configuration. It primarily acts as a manifest, importing all the modular program configurations.
*   **Modular Programs:** The `programs/` directory contains individual `.nix` files for configuring specific applications (e.g., `git.nix`, `hyprland.nix`, `zellij.nix`). This makes it easy to manage, add, or remove applications.
*   **Adding New Programs:** To add a new program, create a `.nix` file in the `programs/` directory defining its configuration and then import it into `home.nix`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustSteveKing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JustSteveKing)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
