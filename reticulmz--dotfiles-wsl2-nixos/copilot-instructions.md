## dotfiles-wsl2-nixos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

NixOS + Home Manager dotfiles for a WSL2 (x86_64-linux) environment. Uses Nix flakes for reproducible configuration.

## Commands

```bash
# Home Manager適用（zsh関数として定義済み）
hms reticulmz

# NixOS適用
sudo nixos-rebuild switch --flake ~/dotfiles#reticulmz

# flake.lockの更新（特定input）
nix flake update <input-name>

# シークレットの編集（sops + age）
sops secrets/home-manager.yaml
```

## Architecture

```
flake.nix                    # エントリーポイント: inputs定義 + NixOS/HM outputs
├── nixos/
│   ├── configuration.nix    # NixOSエントリーポイント + システムパッケージ
│   └── modules/             # wsl, fonts, docker, zsh, nix-ld, nix, locale
└── home-manager/
    ├── home.nix             # HMエントリーポイント + sops設定
    └── programs/            # git, zsh/, direnv, ssh, claude, ide, dev-tools
```

- **NixOS** (`nixosConfigurations."reticulmz"`): システムレベル設定（WSL、フォント、Docker、nix-ld等）
- **Home Manager** (`homeConfigurations."reticulmz"`): ユーザーレベル設定（シェル、Git、IDE、開発ツール等）
- 両者は独立してビルド・適用される

## Secrets

sops + ageで管理。暗号化設定は `.sops.yaml`、暗号化ファイルは `secrets/` 配下。復号には `~/.config/sops/age/keys.txt` が必要。

## Conventions

- モジュールは機能単位で分割（例: `docker.nix`, `git.nix`）
- 機能に直結するパッケージはそのモジュール内に配置（例: `wslu` は `wsl.nix` に）
- 汎用システムパッケージは `configuration.nix` の `environment.systemPackages` に配置
- HM側の開発ツールは `dev-tools.nix` に集約
- 日本語で記述・コミュニケーション

---
> Source: [Reticulmz/dotfiles-wsl2-nixos](https://github.com/Reticulmz/dotfiles-wsl2-nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
