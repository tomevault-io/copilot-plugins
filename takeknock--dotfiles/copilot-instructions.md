## dotfiles

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**重要**: このリポジトリで作業する際は、すべてのドキュメントとコメントを日本語で記載してください。

## リポジトリ概要

開発ツールの設定ファイルを含む個人用dotfilesリポジトリです。主に以下の設定が含まれています：
- シェル環境（Bash、Zsh）
- テキストエディタ（Vim、Emacs）
- PythonとGoの開発環境

## 主要コマンド

### リポジトリセットアップ
```bash
# すべてのdotfilesのシンボリックリンクを作成
./makesymbolic.sh
```

`makesymbolic.sh`スクリプトは、このリポジトリのdotfilesからホームディレクトリへのシンボリックリンクを作成します（`.git`と`..`ディレクトリを除く）。

### シェル固有のコマンド

**Bashエイリアス** (`.bashrc`で定義):
- `apache2ctl` - sudo権限でApacheサーバーを制御
- `mysql5ctl` - sudo権限でMySQLサーバーを制御
- `topcoder` - TopCoderコンテストアプレットを起動
- `codevs` - CodeVSクライアントを起動

**Zshエイリアス** (`.zshrc`で定義):
- `yolo` - `claude --dangerously-skip-permissions`のショートカット

### Vim開発
- Pythonファイルで`Ctrl+E` - 現在のPythonファイルを実行（.vimrc:44で設定）

## アーキテクチャと構造

### 設定ファイル
- **`.bashrc`** - MacPorts、MySQL、Xcode開発者ツール、カスタムバイナリのPATHエクスポートを含む
- **`.zshrc`** - プロンプトでのGitブランチ表示とnpmグローバルパスを含む最小構成
- **`.vimrc`** - dein.vimプラグインマネージャーを使用、プラグイン設定は別のTOMLファイルで管理
  - プラグイン設定の想定パス: `~/.vim/dein/userconfig/plugins.toml`と`plugins_lazy.toml`
  - Python固有の設定の想定パス: `.vim/ftplugin/python.vim`

### パス設定
macOS環境向けに設定されています：
- MacPortsインストール（`/opt/local/`）
- Xcode開発者ツール
- カスタムユーザーバイナリ（`~/bin/`）
- npmグローバルパッケージ（`~/.npm-global/`）

### 重要な注意事項
- `.vimrc`にハードコードされたパス`/home/takeknock/`は調整が必要な場合があります
- Emacs設定ディレクトリ（`.emacs.d/`）はREADMEで参照されていますが、リポジトリには存在しません
- Vimプラグインディレクトリ（`.vim/`）は参照されていますが、リポジトリには存在しません

## このリポジトリでの作業

dotfilesを変更する際：
1. このリポジトリ内のファイルを直接編集
2. `./makesymbolic.sh`を実行してシンボリックリンクを更新
3. 新しいシェルセッションは更新された設定を使用
4. Vimの変更の場合、Vimを再起動するか設定をソース

リポジトリはシンプルな構造で、すべてのdotfilesがルートレベルにあり、ホームディレクトリにシンボリックリンクされます。

## 日本語環境での作業

このリポジトリで作業する際は：
- すべてのコメントとドキュメントを日本語で記載してください
- コミットメッセージも日本語で記載してください
- 既存の日本語コメントのスタイルに従ってください

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/takeknock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
