## dotfiles

> [chezmoi](https://www.chezmoi.io/)でmacOSとdebian系Linuxの開発環境を宣言的に構築する個人用dotfiles。

# dotfiles for cffnpwr

## 概要

[chezmoi](https://www.chezmoi.io/)でmacOSとdebian系Linuxの開発環境を宣言的に構築する個人用dotfiles。
全設定はchezmoiソースで一元管理し、`chezmoi apply`でホームへ反映する。

設計の詳細は [設計ドキュメント](docs/design-doc/README.md) を参照する。設計判断・規約の根拠はそこにある。

## ディレクトリ構成

- `homedir/`: chezmoiソースルート（`.chezmoiroot`で指定）。ホームへ反映する対象はここに限定。
  - `homedir/.chezmoi.toml.tmpl`: apply時の変数定義
  - `homedir/.chezmoidata/`: OS別パッケージなどの宣言データ
  - `homedir/.chezmoiscripts/`: ブートストラップスクリプト
  - `homedir/.chezmoitemplates/`: 共有テンプレート
  - `homedir/dot_config/`: 各種アプリ設定
- `docs/design-doc/`: 設計ドキュメント
- リポジトリ直下: `README.md`・`LICENSE` などソース外のメタ情報

## 編集と反映

chezmoi運用にはこのリポジトリ固有の罠がある。ファイルの編集・diff・apply・反映を行う前に [chezmoiスキル](.claude/skills/chezmoi/SKILL.md) を必ず読む。要点:

- **このリポジトリ（開発clone）とchezmoiソース（`~/.local/share/chezmoi`）は別の場所にある。** 開発cloneの変更をそのままapplyできず、`chezmoi diff`が空になる。反映時は `--source <repo root>` を渡すか、commit & push後にソース側で取り込む。
- **全体applyは `.chezmoiscripts` を実行する。** homebrew/パッケージ導入などの重い副作用がある。特定ファイルだけ反映したい時はtarget pathで範囲を絞る。
- chezmoiにbuild/test/lintのCIはない。検証は `chezmoi diff --source <repo root>` で差分を確認する。

## バージョン管理

- **mainへの直接pushを許可する。** 軽微な更新が多いため、フィーチャーブランチ・PRを必須にしない。

## 規約・制約

- **単一情報源**: 設定はソース（`homedir/`）を編集して反映する。ホームを直接編集しない（applyで上書きされて失われる）。
- **冪等な再構築**: 環境構築は何度実行しても同じ構成へ収束する手続きとして表現する。
- **OS別ネイティブなパッケージ導入**: 抽象化レイヤを挟まず、各OSがネイティブに備える手段で導入する。
- **シークレットの暗号化**: リポジトリに含める秘密情報は暗号化し、平文では置かない。
- ソース内ファイルの命名は [chezmoiの仕様](https://www.chezmoi.io/reference/source-state-attributes/) に従う。
- ブートストラップスクリプトの分類（`run_`/`run_onchange`）と採番・実行順序は [ブートストラップ設計](docs/design-doc/bootstrap.md) に従う。

---
> Source: [cffnpwr/dotfiles](https://github.com/cffnpwr/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
