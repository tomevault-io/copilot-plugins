## fluffy

> NixOS flake configuration for the fluffy server.

# Fluffy

NixOS flake configuration for the fluffy server.

## Development

Enter dev shell: `nix develop`

Available scripts:
- `format` - Format all Nix files
- `lint` - Check for issues (nixf-diagnose, statix, deadnix)
- `fix` - Auto-fix linting issues and format

## Naming Conventions

All names in Nix code must follow:
- **Local let bindings, function args**: camelCase
- **Package names**: kebab-case
- **NixOS options**: kebab-case

## Project Structure

- `flake.nix` - Flake definition with devShell and NixOS configurations
- `configuration.nix` - Main NixOS configuration
- `modules/` - NixOS modules
- `containers/` - Container definitions (quadlet-nix)
- `scripts.nix` - Development scripts (format, lint, fix)

## Hints

- The dashboard is powered by Podfather, which discovers containers via labels. When adding a new container, use `containerLib.podfatherLabels { ... }` in `containerConfig.labels` (see `containers/lib.nix`). For non-container services, register them via `fluffy.podfather.external-apps` in `modules/podfather.nix`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jo-m)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jo-m)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
