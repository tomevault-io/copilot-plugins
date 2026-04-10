## nix-config

> Guidelines for AI agents working in this nix-config repository.

# AGENTS.md

Guidelines for AI agents working in this nix-config repository.

## Repository structure

- **`flake.nix`** — Flake definition with 15 inputs (nixpkgs 25.11, home-manager, denix, nix-darwin, nixos-wsl, etc.)
- **`hosts/`** — 9 host configs using `delib.host` (NixOS desktops: cottonwood, redwood, sequoia, myrtle; NixOS server: bristlecone; WSL: mistletoe; Darwin: lemon, pineapple; installer)
- **`modules/`** — Shared modules using `delib.module` with `singleEnableOption` pattern, organized by category (config, core, desktop, applications, programming, terminal, editors, fonts, services)
- **`config/`** — Static app config files (awesome, niri, kando)
- **`lib/`** — Shared Nix utilities (python-core-packages.nix, xonsh-extra-packages.nix)
- **`packages/`** — Custom packages (rofi-desktop, xxh)
- **`hardware/`** — Hardware-specific configs for desktop hosts
- **`templates/`** — 11 devenv project templates
- **`docs/`** — Documentation (MODULES.md, SETUP.md, TEMPLATES.md, architecture diagrams)

## Verify the target first

- Before making changes, identify the exact target config and the machine/OS required to validate it.
- Do not assume the edited path name matches the flake output or the machine that must run the verification command.
- State this explicitly before editing: `Target config: <name>. Validation host: <machine/OS>. Planned verification: <command>.`

### Host-to-system mapping

| Host | System | Type | Flake output |
|------|--------|------|-------------|
| bristlecone | x86_64-linux | server | `nixosConfigurations.bristlecone` |
| cottonwood | x86_64-linux | desktop | `nixosConfigurations.cottonwood` |
| redwood | x86_64-linux | desktop | `nixosConfigurations.redwood` |
| sequoia | x86_64-linux | desktop | `nixosConfigurations.sequoia` |
| myrtle | x86_64-linux | desktop | `nixosConfigurations.myrtle` |
| mistletoe | x86_64-linux | wsl | `nixosConfigurations.mistletoe` |
| lemon | aarch64-darwin | darwin | `darwinConfigurations.lemon` |
| pineapple | aarch64-darwin | darwin | `darwinConfigurations.pineapple` |
| installer | x86_64-linux | installer | `nixosConfigurations.installer` |

## Always validate with eval

- After every Nix change, run a matching `nix eval` against the exact target output before claiming success.
- Preferred eval targets:
  - NixOS: `nix eval .#nixosConfigurations.<host>.config.system.build.toplevel.drvPath`
  - Darwin: `nix eval .#darwinConfigurations.<host>.config.system.build.toplevel.drvPath`
  - Home Manager: `nix eval .#homeConfigurations.<config>.activationPackage.drvPath`
- When practical, follow eval with the matching dry-run/build/test command. Eval is the minimum bar, not the whole test plan.
- If editing a module used by multiple hosts/platforms, eval at least one NixOS and one Darwin target (if applicable).

## Prefer upstream fixes

- If an issue appears to come from upstream, first check whether it is already fixed upstream.
- This repo uses flakes — update the relevant flake input or `flake.lock` first (`nix flake update <input>` or `nix flake lock --update-input <input>`).
- Prefer an upstream PR commit or a commit already merged upstream before writing a local patch.
- Only write a local patch when those options fail. Keep it minimal.

## Denix module conventions

This repo uses the [denix](https://github.com/yunfachi/denix) framework. All modules follow a consistent pattern:

### Module template

```nix
{ delib, lib, pkgs, inputs, ... }:
let
  sharedPackages = with pkgs; [ ... ];
  linuxOnlyPackages = with pkgs; [ ... ];
in
delib.module {
  name = "category.name";
  options = delib.singleEnableOption false;

  myconfig.always = { myconfig, ... }: {
    # Auto-enable from parent module
    module.name.enable = lib.mkDefault (myconfig.parent.enable or false);
  };

  nixos.ifEnabled = {
    environment.systemPackages = sharedPackages ++ linuxOnlyPackages;
  };

  darwin.ifEnabled = {
    environment.systemPackages = sharedPackages;
  };

  home.ifEnabled = { ... };
}
```

### Key rules

- Use `delib.module` for all shared modules, `delib.host` for host configs.
- Use `delib.singleEnableOption` for feature toggles (default `false` unless always-on).
- Config modules (constants, home, user, overlays, darwin) have no enable toggle — they are always active.
- Route config by block: `nixos.*` for NixOS-only, `darwin.*` for macOS-only, `home.*` for both (via home-manager), `myconfig.*` for pre-evaluation defaults.
- Use `lib.mkDefault` for values that hosts should be able to override.
- Split packages into `sharedPackages`, `linuxOnlyPackages`, `darwinOnlyPackages` when a module targets multiple platforms.

### Host template

```nix
{ delib, pkgs, inputs, lib, ... }:
delib.host {
  name = "hostname";
  type = "desktop";  # desktop | wsl | darwin | installer | server
  system = "x86_64-linux";  # or aarch64-darwin

  home.home.stateVersion = "24.05";

  nixos = {
    system.stateVersion = "25.11";
    imports = [ ../../hardware/hostname.nix ];
  };

  myconfig = {
    constants.username = "nano";
    modules.feature.enable = true;
  };
}
```

## Package and overlay patterns

- Unstable packages are pinned via overlays in `modules/config/overlays.nix` — add new unstable pins there, not inline.
- Custom packages live in `packages/` and are added to the overlay.
- Shared Python package lists live in `lib/python-core-packages.nix` and `lib/xonsh-extra-packages.nix`.

## Cross-platform awareness

- This repo targets NixOS (x86_64-linux), Darwin (aarch64-darwin), and WSL.
- When editing a module, check whether it uses `nixos.*`, `darwin.*`, or both — ensure changes work on all applicable platforms.
- Darwin uses Homebrew casks for GUI apps (configured in `modules/config/darwin.nix`). NixOS installs GUI apps via `environment.systemPackages`.
- macOS hosts use `paneru` for tiling; NixOS desktops use `niri` (Wayland) or `awesome` (X11).

## Naming conventions

- **Hosts**: Named after trees/plants (bristlecone, redwood, lemon, mistletoe, etc.)
- **Modules**: Hierarchical dot notation (`desktop.niri`, `programming.cloud`, `applications.creative`)
- **Commit messages**: Short imperative style — "added opencode", "fixed autohide", "changed kitty to accept alt"

## What NOT to do

- Do not create NixOS options when a simple `environment.systemPackages` addition suffices.
- Do not hardcode usernames — use `myconfig.constants.username`.
- Do not add packages to `modules/core/default.nix` unless they should be on every single host. Use feature modules instead.
- Do not manage secrets (API keys, tokens, SSH keys) through Nix — they are handled outside this repo.
- Do not modify `config/awesome/lain` or `config/awesome/freedesktop` — these are git submodules.
- Do not add Claude Code settings to home-manager — `~/.claude/settings.json` must remain writable at runtime.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rft)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rft)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
