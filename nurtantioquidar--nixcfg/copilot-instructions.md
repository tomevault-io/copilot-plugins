## nixcfg

> This repository manages personal Nix configuration for macOS and WSL.

# Agent Guide

This repository manages personal Nix configuration for macOS and WSL.

## Scope

- macOS host: `darwinConfigurations.styx` with IT hostname alias `darwinConfigurations.MAC-F0Q3XN9HR9`
- WSL host: `nixosConfigurations.wsl`
- Standalone macOS Home Manager profile: `homeConfigurations.hades`
- Shared Home Manager profile: `nix/home/home.nix`
- Primary reference docs: `docs/setup-guide.md`

## Working Rules

- Prefer the existing Nix module layout over introducing new structure.
- Keep host-specific changes under `nix/hosts/mbp` or `nix/hosts/wsl`.
- Keep host-specific helper scripts under `nix/hosts/<host>/scripts`.
- Keep shared user packages, shells, Git, prompt, and dotfile behavior under `nix/home`.
- Keep shared host import wiring in `nix/lib/mkImports.nix`; prefer updating host module lists over bypassing the helper.
- Keep custom package overrides in `nix/overlays` and reusable local package definitions in `nix/packages`; expose new packages through `nix/overlays/default.nix`.
- Keep Codex CLI user-managed through `nix/home/codex.nix`; use `codex-upgrade` to rerun OpenAI's standalone installer without sudo.
- Keep global npm package management in `nix/home/node-packages.nix`. That module writes `~/.npmrc` so `npm install --global` uses the user-writable `~/.local` prefix instead of the immutable Nix store.
- Keep Ghostty user-managed through `nix/home/ghostty.nix`.
- Keep Zellij user-managed through `nix/home/zellij.nix`. Its wrapper intentionally normalizes `TMPDIR` outside direnv/Nix `nix-shell.*` temp directories, sets `ZELLIJ_SOCKET_DIR` to a short per-user `/tmp` path to avoid macOS socket path limits, and downgrades Ghostty's outer `TERM` to avoid leaked DSR responses like `?997;2n` when launching Zellij from this repo. Ghostty config sets the left Option key as terminal Alt for Zellij bindings while the right Option key remains available for macOS character input. Zellij clears default bindings so `Alt+Left`/`Alt+Right` stay available for shell word navigation, and `nix/home/zsh.nix` binds the common Option+Arrow escape sequences to zsh word movement so trailing `C`/`D` bytes are not inserted. `Alt+Shift+f` toggles floating panes, `Alt+Shift+n` opens a tab, and `Ctrl+y` launches zellij-forgot. The zellij-autolock plugin is defined but intentionally not loaded because upstream issue fresh2dev/zellij-autolock#20 reports that it can immediately undo manual `Ctrl+g` lock/unlock changes. For Zellij prompts that show `<Del>` on Mac keyboards, use `Fn+Delete`; Ghostty cannot bind `fn` directly.
- Keep the VS Code CLI user-scoped on macOS. Home Manager installs a `code` wrapper for `/Users/hades/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code` because this machine previously had VS Code ownership and app-bundle issues when moving between Homebrew, Home Manager app links, and system locations.
- Keep local secrets outside this flake. The expected external secrets path is documented in `docs/setup-guide.md`.
- Git SSH signing verification is managed through Home Manager. `nix/home/git.nix` writes `~/.config/git/allowed_signers` from the external `userEmail` and `sshSigningKey` values when 1Password signing is enabled.
- Do not mutate Nix store paths or Nix-managed Homebrew tap symlinks directly. Change flake inputs or Nix modules instead.
- Use `rg` for repository search.
- Use `nixpkgs-fmt` for Nix formatting when editing Nix files.
- Markdown documentation filenames must be lower kebab case. `AGENTS.md` is the only exception.

## Build And Validation

Run lightweight evaluation before handing off meaningful config changes:

```bash
nix --extra-experimental-features nix-command --extra-experimental-features flakes flake check --no-build
```

For the Darwin host, select the configured host explicitly:

```bash
sudo darwin-rebuild switch --flake /Users/hades/.config/nix#styx --impure
```

The IT-managed macOS hostname is also exposed as `#MAC-F0Q3XN9HR9`; keep `#styx` as the friendly alias.

For user-level macOS Home Manager changes, validate the standalone profile without sudo:

```bash
nix --extra-experimental-features nix-command --extra-experimental-features flakes build /Users/hades/.config/nix#homeConfigurations.hades.activationPackage --impure
```

Activate the standalone profile only when the change should be applied to the user environment:

```bash
home-manager switch --extra-experimental-features nix-command --extra-experimental-features flakes --flake /Users/hades/.config/nix#hades --impure
```

Keep root-required macOS settings in `darwinConfigurations.styx` / `darwinConfigurations.MAC-F0Q3XN9HR9`; move user packages, shells, Git, prompt, and dotfile behavior through `homeConfigurations.hades` when possible.

For WSL:

```bash
sudo nixos-rebuild switch --flake ~/.config/nix#wsl
```

## Homebrew Notes

Homebrew bootstrap, pinned taps, and privileged casks are managed through `nix-homebrew` and `nix/hosts/mbp/homebrew.nix`.

- Keep `homebrew/cask` in `homebrew.taps`.
- Keep `inputs.homebrew-cask` exposed through `nix-homebrew.taps."homebrew/homebrew-cask"`.
- Keep `manaflow-ai/cmux` in `homebrew.taps` and expose `inputs.homebrew-cmux` through `nix-homebrew.taps."manaflow-ai/homebrew-cmux"` for the cmux cask.
- Keep ordinary app-bundle casks out of Darwin `homebrew.casks`; declare them in `nix/home/homebrew.nix` instead. Home Manager writes a user Brewfile and runs `brew bundle install --no-upgrade` with `HOMEBREW_CASK_OPTS=--appdir=/Users/hades/Applications`. Casks with package installers or privileged components may still need the admin path.
- Do not enable automatic `brew bundle cleanup` in the user Homebrew module; cleanup sees all Homebrew casks, including privileged casks owned by the Darwin profile.
- Keep `homebrew.onActivation.cleanup = "none"` unless intentionally pruning user-installed Homebrew apps.
- Treat cask ownership changes as state-changing: removing a cask from the user-managed list can uninstall it during Home Manager activation unless it is explicitly exempted or moved through an approved admin/user flow. For audit-only work, report drift without changing cask membership.
- Activation sets `HOMEBREW_NO_INSTALL_FROM_API=1`, so cask behavior should be checked with the no-API path when debugging casks.
- If a cask DSL error appears, consider whether `nix-homebrew`, its `brew-src`, and `homebrew-cask` are pinned to compatible revisions.

## Documentation Maintenance

Update this file whenever a change meaningfully alters how future agents should work in this repo.

A meaningful change includes:

- adding, removing, or renaming flake outputs, hosts, modules, or major directories;
- changing rebuild, validation, formatting, or activation commands;
- changing Homebrew management, cask resolution, or cleanup behavior;
- changing where secrets, dotfiles, shells, Git signing, or shared Home Manager configuration live;
- adding a new recurring troubleshooting rule or operational pitfall.
- changing documentation naming conventions.

Small package list edits, version bumps, and lock-file refreshes usually do not need an `AGENTS.md` update unless they change the workflow above.

---
> Source: [nurtantioquidar/nixcfg](https://github.com/nurtantioquidar/nixcfg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
