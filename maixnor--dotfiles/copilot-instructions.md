## dotfiles

> This repository serves as a comprehensive collection of NixOS and Home Manager configurations, managed as a Nix flake. It's designed to define and deploy reproducible system-wide and user-specific environments across multiple machines (e.g., `bierbasis`, `bierzelt`, `wieselburg`). The configurations cover a wide range of functionalities, from core system services and desktop environments to development toolchains and self-hosted applications.

# GEMINI.md - Project Overview

This repository serves as a comprehensive collection of NixOS and Home Manager configurations, managed as a Nix flake. It's designed to define and deploy reproducible system-wide and user-specific environments across multiple machines (e.g., `bierbasis`, `bierzelt`, `wieselburg`). The configurations cover a wide range of functionalities, from core system services and desktop environments to development toolchains and self-hosted applications.

`bierzelt` is a Lenovo Laptop. `bierbasis` is a tower PC currently not in use, no need to update that. `wieselburg` is a VPS server.

## Project Structure and Purpose

The project leverages Nix flakes to ensure consistent and declarative system management. Key aspects include:
- **Modular Design:** Configurations are organized into reusable modules (`modules/`, `services/`) for clarity and maintainability.
- **Machine-Specific Configurations:** Dedicated directories (`bierbasis/`, `bierzelt/`, `wieselburg/`) house the unique configurations for different physical and virtual machines.
- **Development Environment:** Comprehensive development setups are defined, including tools for Rust, Python, Node.js, R, various IDEs, and terminal enhancements like `tmux`.
- **Self-Hosted Services:** Management of applications such as Immich (photo management), PostgreSQL, and Redis, often integrated with reverse proxies like Traefik.
- **Reproducibility:** The use of flakes pins dependencies, ensuring that environments can be recreated consistently.

## Key Technologies

- **NixOS:** A Linux distribution built on Nix, providing declarative and reproducible system configuration.
- **Nixpkgs:** The large collection of packages for Nix.
- **Home Manager:** A tool for managing user-specific configurations declaratively using Nix.
- **Nix Flakes:** A feature of Nix that enables reproducible and discoverable Nix projects.
- **`just` (Justfile):** A command runner used to simplify common tasks like updating configurations and building virtual machines.
- **`tmux`:** A terminal multiplexer with extensive custom configurations, plugins, and keybindings.
- **KDE Plasma:** The desktop environment configured for some systems.
- **`podman` & `libvirtd`:** Containerization and virtualization technologies.
- **PostgreSQL & Redis:** Database and caching services for applications.
- **Traefik:** Used for reverse proxying and SSL termination for self-hosted services.
- **`ollama`:** A local large language model serving platform.

## Building and Running

This project uses Nix flakes for system and user configuration. The primary way to apply configurations is through `nixos-rebuild switch` for system-wide changes and `home-manager switch` for user-specific settings.

For convenience, a `justfile` is provided with common commands:

-   **Update a host's NixOS configuration:**
    ```bash
    just update <hostname>
    # Example: just update bierbasis
    ```
    This command performs a `git pull` and then `sudo nixos-rebuild switch` to apply the latest configuration for the specified host.

-   **Build a VM for a specific host:**
    ```bash
    just vm <hostname>
    # Example: just vm wieselburg
    ```
    This builds a virtual machine using the host's configuration.

-   **Specific host commands (e.g., `bierzelt`, `bierbasis`, `wieselburg`):** These commands often call the `update` command with the appropriate hostname.
    ```bash
    just bierzelt
    just bierbasis
    just wieselburg
    ```

## Development Conventions

-   **Nix Flakes:** All configurations are managed using Nix flakes, ensuring that dependencies are pinned and environments are reproducible.
-   **Modular Configuration:** Configurations are broken down into smaller, reusable `.nix` modules, promoting organization and ease of maintenance.
-   **`nixfmt`:** The `flake.nix` specifies `pkgs.nixfmt` as the formatter, indicating a preference for consistent code formatting within the Nix files.
-   **Secrets Management:** Sensitive information, such as API keys and JWT secrets, is handled through dedicated secrets files (e.g., `/var/lib/immich/secrets.env`), generated via activation scripts where appropriate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maixnor)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maixnor)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
