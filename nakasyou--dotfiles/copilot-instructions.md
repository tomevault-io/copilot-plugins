## dotfiles

> This repository manages NixOS and nix-darwin hosts plus the Home Manager profile for `nakasyou`.

# Repository Guidelines

## Project Structure & Module Organization
This repository manages NixOS and nix-darwin hosts plus the Home Manager profile for `nakasyou`.

- `flake.nix`: main entry point; exposes `nixosConfigurations.p14s` and `darwinConfigurations.mac-mini`.
- `hosts/`: host-specific entry points. Keep generated hardware files under the matching host, e.g. `hosts/p14s/hardware-configuration.nix`.
- `modules/nixos/`: reusable NixOS modules grouped by role such as desktop, audio, input, Docker, and hardware quirks.
- `users/nakasyou/home.nix`: user-level packages, desktop settings, MIME associations, and XDG-managed files.
- `nixos/configuration.nix`: compatibility shim that imports `hosts/p14s`.
- `pkgs/`: custom package definitions and patches.
- `gnome/`: static desktop assets such as `gnome/monitors.xml`.

## Build, Test, and Development Commands
Run commands from the repository root.

- `nix flake show`: verify the flake evaluates and exposes the expected outputs.
- `sudo nixos-rebuild switch --flake .#p14s`: apply the P14s NixOS system and Home Manager changes together.
- `darwin-rebuild switch --flake .#mac-mini`: apply the mac-mini nix-darwin configuration.
- `nix build .#nixosConfigurations.p14s.config.system.build.toplevel`: build the full P14s system closure without switching.
- `nix build .#nixosConfigurations.p14s.config.home-manager.users.nakasyou.home.activationPackage`: validate the Home Manager config only.
- `nix build .#darwinConfigurations.mac-mini.system`: validate the mac-mini nix-darwin configuration.
- `nix-instantiate --eval pkgs/turbowarp-desktop.nix`: quick syntax check for a custom package file.

## Coding Style & Naming Conventions
Use standard Nix formatting: two-space indentation, trailing semicolons, and compact attribute sets. Keep modules declarative and group related options together.

- Prefer lowercase file names with hyphens, e.g. `input-ja-hazkey.nix`.
- Add short comments only where system behavior is non-obvious, such as kernel or device-specific workarounds.
- Reuse `let` bindings for shared values instead of repeating paths or package expressions.
- Keep host files thin; put reusable behavior in `modules/nixos/` and host-specific hardware behavior in `modules/nixos/hardware/`.

## Testing Guidelines
There is no dedicated test suite in this checkout. Treat evaluation and builds as the validation path.

- After structural changes, run `nix flake show`.
- After NixOS module edits, run the relevant `nix build .#nixosConfigurations.p14s...` target before switching.
- After nix-darwin edits, run `nix build .#darwinConfigurations.mac-mini.system`.
- For desktop or service changes, confirm the affected app, service, or setting works after `nixos-rebuild switch`.

## Commit & Pull Request Guidelines
Git history may be sparse in this workspace snapshot, so follow conservative defaults.

- Write commit subjects in imperative mood, e.g. `nixos: tune TrackPoint sensitivity`.
- Keep commits scoped to one concern: system config, home config, darwin config, or package work.
- PRs should explain intent, affected files, rollback risk, and include screenshots when changing GNOME, MIME, or desktop behavior.

## Configuration Notes
Avoid editing generated files casually. `hosts/*/hardware-configuration.nix` should usually change only after hardware reprobe, while user-specific paths and host names should stay consistent with `flake.nix` and `users/nakasyou/home.nix`.

---
> Source: [nakasyou/dotfiles](https://github.com/nakasyou/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
