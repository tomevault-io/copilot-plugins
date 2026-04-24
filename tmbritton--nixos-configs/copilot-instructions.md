## nixos-configs

> This repository contains NixOS configurations for multiple machines using Nix flakes:

# NixOS Configuration Repository

## Repository Overview

This repository contains NixOS configurations for multiple machines using Nix flakes:

### Machines
- **laptop**: Development laptop with power management optimizations
- **desktop**: Desktop workstation with gaming, graphic design, and NVIDIA GPU support

### Common Configuration
- **Architecture**: aarch64-linux (ARM64)
- **Desktop Environment**: GNOME with GDM
- **Primary User**: tom
- **Time Zone**: America/Chicago
- **NixOS Version**: 24.11

This is a development-focused configuration with support for multiple programming languages, development tools, and machine-specific capabilities (gaming, graphic design on desktop; power management on laptop).

## Repository Structure

### File Organization

```
.
├── flake.nix                           # Flake definition with all machine configs
├── flake.lock                          # Flake lock file (auto-generated)
├── common.nix                          # Shared configuration across all machines
├── git.nix                            # Git configuration module (shared)
├── configuration.nix                   # OLD - can be removed
├── hosts/
│   ├── laptop/
│   │   ├── configuration.nix          # Laptop-specific configuration
│   │   └── hardware-configuration.nix # Laptop hardware config (auto-generated)
│   └── desktop/
│       ├── configuration.nix          # Desktop-specific configuration
│       └── hardware-configuration.nix # Desktop hardware config (PLACEHOLDER - needs generation)
└── Claude.md                          # This file
```

### Configuration Architecture

#### Flakes-Based Multi-Machine Setup

This repository uses **Nix flakes** to manage multiple machine configurations:

1. **flake.nix**: Defines all machine configurations (laptop, desktop) and their module dependencies
2. **common.nix**: Contains configuration shared across all machines
3. **git.nix**: Git-specific shared configuration
4. **hosts/[machine]/configuration.nix**: Machine-specific configuration (hostname, packages, services)
5. **hosts/[machine]/hardware-configuration.nix**: Hardware-specific configuration (auto-generated per machine)

#### Module Loading Order

For each machine, modules are loaded in this order:
1. hosts/[machine]/configuration.nix (machine-specific)
2. hosts/[machine]/hardware-configuration.nix (hardware-specific)
3. common.nix (shared configuration)
4. git.nix (git configuration)

### Key Configuration Areas

#### Shared Configuration (common.nix)

All machines share:
- **Bootloader**: systemd-boot with EFI support
- **Networking**: NetworkManager, custom DNS (Cloudflare 1.1.1.1)
- **Desktop**: GNOME with auto-login for user "tom"
- **Audio**: PipeWire (PulseAudio disabled)
- **VM Integration**: Spice guest agent for clipboard sharing
- **Development Environment**: Full toolchain for Node.js, Go, Python, C/C++, Lua
- **Shell Aliases**: clip, rebuild, config, rg, ls
- **Neovim Auto-Setup**: Automatically clones kickstart.nvim on first boot
- **Fonts**: JetBrainsMono Nerd Font

#### Development Environment (Shared)

All machines include toolchains for:
- **Node.js**: v22 with TypeScript and typescript-language-server
- **Go**: Go compiler with gopls LSP
- **Python**: Python 3.12 with pip
- **C/C++**: gcc, gnumake, cmake, pkg-config
- **Lua**: lua with luarocks
- **General Tools**: neovim, ripgrep, fzf, direnv, tmux, tree-sitter, eza

#### Laptop-Specific Configuration (hosts/laptop/configuration.nix)

- **Hostname**: laptop
- **Power Management**: TLP enabled with AC/battery profiles
- **Battery Thresholds**: Charge between 40-80% (if hardware supports)
- **Additional Packages**: powertop, acpi
- **Services**: upower for battery monitoring

#### Desktop-Specific Configuration (hosts/desktop/configuration.nix)

- **Hostname**: desktop
- **GPU**: NVIDIA with proprietary drivers
- **Gaming**: Steam with remote play and dedicated server support, gamemode
- **Graphic Design**: Krita, Scribus, GIMP, Inkscape, Blender
- **Tablet Support**: OpenTablet driver for drawing tablets
- **Additional Tools**: Audacity, nvtop for GPU monitoring
- **OpenGL**: Full 32-bit and 64-bit support for gaming

#### Shell Configuration (common.nix)

Aliases available on all machines:
- `clip`: Copy to clipboard using xclip
- `rebuild`: Run `sudo nixos-rebuild switch --flake .`
- `config`: Edit ~/nixos-configs/ in neovim
- `rg`: ripgrep command
- `ls`: eza with icons

#### Git Configuration (git.nix)

- Default branch: main
- Pull strategy: rebase
- User: Tom Britton (britton.tm@gmail.com)
- Editor: vim
- Custom alias: `git lg` for pretty graph logs

#### SSH Configuration (common.nix)

- SSH agent enabled
- GitHub pre-configured with ~/.ssh/id_ed25519

## Development Conventions

### File Modification Guidelines

1. **Shared Configuration**: Add to `common.nix` for settings that apply to all machines
2. **Machine-Specific**: Add to `hosts/[machine]/configuration.nix` for machine-only settings
3. **DO NOT MODIFY**: Never manually edit hardware-configuration.nix files - they're auto-generated
4. **New Modules**: Create separate .nix files for substantial new functionality and import them in flake.nix
5. **Flake Structure**: All machines must be defined in flake.nix under `nixosConfigurations`

### Module Structure Pattern

When creating new module files, follow this pattern:
```nix
{ config, pkgs, ... }:

{
  # Configuration attributes here
}
```

### Adding Packages

- **Shared packages**: Add to `environment.systemPackages` in `common.nix`
- **Machine-specific packages**: Add to `environment.systemPackages` in `hosts/[machine]/configuration.nix`

### Adding a New Machine

To add a new machine to this repository:

1. Create directory: `mkdir -p hosts/[machine-name]`
2. Generate hardware config on target machine:
   ```bash
   sudo nixos-generate-config --show-hardware-config > ~/nixos-configs/hosts/[machine-name]/hardware-configuration.nix
   ```
3. Create `hosts/[machine-name]/configuration.nix` with machine-specific settings
4. Add machine to `flake.nix` under `nixosConfigurations`:
   ```nix
   [machine-name] = nixpkgs.lib.nixosSystem {
     system = "aarch64-linux";  # or x86_64-linux
     modules = [
       ./hosts/[machine-name]/configuration.nix
       ./hosts/[machine-name]/hardware-configuration.nix
       ./common.nix
       ./git.nix
     ];
   };
   ```

### Configuration Naming Conventions

- Use descriptive names for module files (e.g., `git.nix`, `gaming.nix`)
- Keep module files focused on a single concern
- Use lowercase with hyphens for multi-word filenames
- Machine hostnames should be simple and descriptive (laptop, desktop, server)

## Applying Changes

### On Current Machine (Laptop)

After modifying configuration files:

1. **Test the configuration**:
   ```bash
   sudo nixos-rebuild test --flake .#laptop
   ```

2. **Apply changes**:
   ```bash
   sudo nixos-rebuild switch --flake .#laptop
   # Or use the alias (automatically uses flake):
   rebuild
   ```

3. **Rollback if needed**:
   ```bash
   sudo nixos-rebuild switch --rollback
   ```

### On Desktop (Not Yet Set Up)

When setting up the desktop for the first time:

1. **Clone this repository** on the desktop
2. **Generate hardware configuration**:
   ```bash
   sudo nixos-generate-config --show-hardware-config > ~/nixos-configs/hosts/desktop/hardware-configuration.nix
   ```
3. **Review desktop configuration** and adjust if needed
4. **Apply configuration**:
   ```bash
   cd ~/nixos-configs
   sudo nixos-rebuild switch --flake .#desktop
   ```

### Updating Flake Inputs

To update nixpkgs or other inputs:

```bash
nix flake update
sudo nixos-rebuild switch --flake .#[machine-name]
```

### Building for a Different Machine

You can build any machine's configuration from any machine:

```bash
# Build laptop config from desktop (or vice versa)
nixos-rebuild build --flake .#laptop

# Build desktop config from laptop (or vice versa)
nixos-rebuild build --flake .#desktop
```

## Important Notes for Claude Code

- This repository uses **Nix flakes** for multi-machine management
- **Experimental features** enabled: flakes, nix-command
- **Unfree packages** allowed (`nixpkgs.config.allowUnfree = true`)
- **Current machine**: laptop (VM environment with SPICE integration)
- **Desktop machine**: Configuration ready but hardware-configuration.nix needs generation on actual hardware
- User "tom" has sudo access (member of "wheel" group)
- Natural scrolling enabled for mouse and touchpad
- The `rebuild` alias automatically uses the flake configuration

### Machine-Specific Considerations

**Laptop**:
- Power management via TLP
- Battery optimization enabled
- Lighter package set

**Desktop**:
- NVIDIA proprietary drivers
- Gaming-optimized (Steam, gamemode)
- Creative suite (Krita, Scribus, GIMP, Inkscape, Blender)
- OpenTablet driver for graphics tablets
- Performance-focused configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmbritton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
