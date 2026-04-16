## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

chezmoiで管理されるdotfilesリポジトリ。クロスプラットフォーム（Linux/macOS/Windows）対応。

## chezmoi コマンド

```bash
chezmoi apply           # 変更を適用
chezmoi diff            # 差分を確認
chezmoi add ~/.config/x # ファイルを追加
chezmoi add --template ~/.config/x  # テンプレートとして追加
```

## ファイル命名規則

- `dot_` → `.` に変換（例: `dot_zshrc` → `.zshrc`）
- `.tmpl` → テンプレートファイル（Go template構文）
- `executable_` → 実行権限付き
- `private_` → パーミッション 0600

## テンプレート構文

- OS判定: `{{ .chezmoi.os }}` （darwin, linux, windows）
- WSL判定: `{{ .chezmoi.kernel.osrelease | lower | contains "microsoft" }}`
- 条件分岐: `{{- if eq .chezmoi.os "darwin" }}...{{- end }}`

## 注意事項

`.chezmoi.toml.tmpl` で `autoCommit` と `autoPush` が有効。`chezmoi apply` 実行時に自動コミット・プッシュされる。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rizumi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
