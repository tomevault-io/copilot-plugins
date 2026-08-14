## ow1cfg-nix

> NixOS/home-manager flake configuration using `flake-parts` + `import-tree`.

# AGENTS.md

NixOS/home-manager flake configuration using `flake-parts` + `import-tree`.

## Building

```bash
# Build switch to host
sudo nixos-rebuild switch --flake .#ow1top

# Build home-manager only
home-manager switch --flake .
```

## Key commands

```bash
# Format Nix files
alejandra --check .

# Dev shell (formatter + nixd + lua-language-server)
nix develop
```

## Structure

- `flake.nix` - entry point, uses `import-tree` to discover modules
- `modules/` - flake modules (e.g., `neovim/default.nix` exports a package)
- `hosts/` - host-specific NixOS configs (defines `nixosModules.*`)
- `home/` - home-manager user configs (imported by hosts)

## Important quirks

- `import-tree ./modules` recursively imports all `.nix` files as flake modules - no explicit list needed
- The neovim module exports both a package (`self.packages.x86_64-linux.neovim`) and a nixosModule
- Hosts reference modules via `self.nixosModules.*` (not `inputs.*`)
- Timezone: Europe/Copenhagen, keymap: dk-latin1 (Danish locale)

---
> Source: [Ow1Dev/ow1cfg.nix](https://github.com/Ow1Dev/ow1cfg.nix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
