## hyper-modern-nix

> - `nix-shell` - Enter development environment

# NixOS Configuration Guidelines

## Build Commands

- `nix-shell` - Enter development environment
- `home-manager switch` - Apply home configuration
- `nixos-rebuild switch` - Apply system configuration changes
- `nix flake update` - Update flake inputs
- `sudo nixos-rebuild switch --flake .#watchtower` - Update specific host
- `treefmt` - Format code across the repository
  TODO[b7r6]: we may want to use treefmt, depends on emacs integration
- `./install-hyper-modern.sh` - Install dotfiles with automatic backup

## Nix Development

- `nixd` - Preferred Nix language server (not nil or rnix-lsp)
- `nixpkgs-fmt` - Standard formatter for Nix files
- Ensure proper Emacs integration via eglot/lsp-mode

## Python Tools (Astral)

- `uv` - Package manager for Python (preferred over pip)
- `uv2nix` - Generate Nix expressions from Python requirements
- `ruff` - All-in-one Python linter and formatter (preferred)
- Interactive usage encouraged for development workflows

## Stylix Theme Configuration

- **Base16 Palette**:

  - Dark background: `#101216`
  - Selection background: `#161B22`
  - Comments/secondary: `#5C6370`
  - Primary text: `#bababa`
  - Red: `#f78166`
  - Orange: `#f49b4f`
  - Yellow: `#FDA656`
  - Green: `#8ddb8c`
  - Cyan: `#96cffe`
  - Blue: `#539bf5`
  - Magenta: `#d5b7f4`
  - Purple: `#AB5DFF`

- **Terminal Compatibility**:
  - Support both ANSI and true-color modes
  - Primary terminals: ghostty, wezterm
  - tmux configured with RGB support
  - Default fonts: JetBrains Mono, Inter
  - No transparency (opacity: 1.0)

## User Management

- Structured as multi-user from the start
- Core users: `b7r6` and `maskirov`
- Modular approach:
  - Common CLI tools in `modules/common/cli`
  - User defaults in `modules/common/users/default`
  - User-specific configs in `modules/common/users/{username}`
  - System users defined in `hosts/nixos-common.nix`
- Default tools include:
  - wezterm/ghostty for terminal
  - neovim with LSP support
  - uv/ruff for Python development
  - tmux configured for modern terminals

## Repository Structure

- `flake.nix` - Main configuration entrypoint
- `modules/` - Shared configuration modules
  - `modules/common/` - Cross-platform shared settings
    - `modules/common/cli/` - Common command-line tools
    - `modules/common/users/` - Home Manager user configurations
  - `modules/nixos/` - NixOS-specific flake outputs
  - `modules/darwin/` - macOS-specific flake outputs
  - `modules/stylix/` - Theme configuration (imported by hosts)
  - `modules/home/` - Home-manager specific modules
- `hosts/` - Host-specific configurations
  - `hosts/nixos-common.nix` - Shared NixOS settings (users, sudo, etc)
  - `hosts/default.nix` - Host definitions
  - `hosts/watchtower/` - Example aarch64-linux VM
- `legacy/` - Reference configurations from previous setup

## Flake Structure Notes

- System-specific settings (users.users, security.sudo) belong in hosts/nixos-common.nix
- User configurations for Home Manager are in modules/common/users/\*
- Stylix theme is a standalone module imported by host configurations
- modules/nixos/default.nix only defines flake outputs
- Each module should only define options valid in its context
- System modules should be directly imported by hosts, not via flake-parts

## Code Style

- **Indentation**: 2 spaces
- **Max Line Length**: 80 columns
- **Formatting Tools**:
  - Nix: `nixpkgs-fmt` (M-z in nix-mode)
  - Python: `ruff format` (exclusively)
  - Java: `clang-format`
  - Starlark/Bazel: `starlark-format`
  - Kotlin: `ktlint`
  - Swift: `swift-format`
  - TypeScript/JS: `prettier`
  - Shell: `shfmt`

## Key Features

- Multi-platform support (x86_64/aarch64, Linux/Darwin)
- Multi-user support with shared baseline configuration
- Home Manager integration with declarative package management
- Emacs setup with format-all-mode and AI assistance
- Modern terminal integration via ghostty/wezterm
- Custom shell utilities and keyboard shortcuts
- Astral Python toolchain (`uv` and `ruff`)
- Modern Nix development with `nixd`
- Consistent theming via Stylix

---
> Source: [hyper-modern-dx/hyper-modern-nix](https://github.com/hyper-modern-dx/hyper-modern-nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
