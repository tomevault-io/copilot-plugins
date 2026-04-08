## nixcfg

> This file provides context and rules for AI agents (like GitHub Copilot,

# AI Agent Instructions

This file provides context and rules for AI agents (like GitHub Copilot,
ChatGPT, etc.) interacting with this repository.

## Operational Rules

1. **Format Code**: Always run `nix fmt` after modifying `.nix` files.
2. **Verify Changes**:
   - Use `nixos-rebuild test --flake .#<host>` to verify system changes before
     committing (if running in a suitable environment).
   - Use `nix flake check` to verify flake integrity.
3. **Respect Conventions**:
   - Use the `__cfg` namespace for new options.
   - Do not hardcode paths or users unless absolutely necessary; use variables.
4. **Security**:
   - Never commit plaintext secrets.
   - Always check if a file is a SOPS secret file before reading or editing
     (look for `.sops.yaml` or `.yaml` files with encrypted data).

## Tooling

- **Flakes**: This is a pure flake setup. All commands should use `--flake .`.
- **Custom Packages**: Defined in `packages/` and overlaid onto `pkgs`. Check `packages/<package-name>/default.nix`.
- **Disko**: Used for disk partitioning.
- **Home Manager**: Used as a NixOS module.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suicide)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/suicide)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
