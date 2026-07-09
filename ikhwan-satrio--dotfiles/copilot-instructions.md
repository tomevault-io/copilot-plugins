## dotfiles

> sudo nixos-rebuild switch --flake .#nixos-btw --impure

# AGENTS.md

## Build & switch

```bash
# Main rebuild (also available as `ns` alias in fish)
sudo nixos-rebuild switch --flake .#nixos-btw --impure

# Validate
nix flake check

# Enter dev shell
nix develop
```

## Structure

- **`host/nixos-btw/configuration.nix`** — single NixOS config (one host: `nixos-btw`)
- **`host/nixos-btw/root-modules/`** — NixOS-level modules (intel graphics, grub, starship, smooth-scroll)
- **`host/nixos-btw/home-manager/modules/`** — user-level home-manager modules (fish, browsers, gtk, xdg, vesktop, vscode, noctalia)
- **`host/pkgs/`** — custom package expressions (cockpit-machines, free-download-manager, smooth-scroll)
- **`config/`** — static dotfiles (kitty, starship, tmux, fastfetch, neofetch) — *not* nix-managed; ~/stow or manual copy
- **`vesktop-themes/`** — imported into home-manager via `home.file`

## Quirks

- `home-manager.backupFileExtension = "hm-backup"` — existing files get renamed with `.hm-backup` suffix on home-manager apply
- Flake requires `--impure` for nixos-rebuild (uses `/etc/nixos/hardware-configuration.nix` + unfree)
- Gitignore only covers `.git-credentials` and `.gitconfig` — be careful with secrets
- No tests, no CI, no formatter/lint config in repo
- `nixpkgs` uses `nixos-unstable` branch; `allowUnfree = true`
- CachyOS kernel, Hyprland (Wayland), SDDM display manager, Intel Alder Lake GPU
- Timezone `Asia/Jakarta`, locale `en_US.UTF-8` with Indonesian secondary locale settings
- User `wanto`, state version `26.05`

---
> Source: [ikhwan-satrio/dotfiles](https://github.com/ikhwan-satrio/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
