## dotfiles

> このリポジトリは、開発環境を統一し、効率化するためのdotfilesを管理します。

# GEMINI.md

このリポジトリは、開発環境を統一し、効率化するためのdotfilesを管理します。
macOS環境を想定しており、Fishシェル、Neovim、Tmux、Gitなどの設定が含まれています。

## 概要

このdotfilesは、以下の主要なツールを設定します。

- **シェル:** Fish (with Tide prompt)
- **エディタ:** Neovim (LazyVimベース)
- **ターミナルマルチプレクサ:** Tmux (gpakosz/.tmuxベース)
- **Git:** カスタムエイリアスとCommitizenによるコミット規約
- **パッケージ管理:** Homebrew (Brewfileによる一括管理)
- **その他:** Lazygit, Karabiner-Elementsなど

## セットアップ

1. **リポジトリのクローン:**

    ```bash
    git clone https://github.com/bear29ers/dotfiles.git ~/dotfiles
    ```

2. **シンボリックリンクの作成:**
    セットアップスクリプトを実行して、ホームディレクトリに必要な設定ファイルのシンボリックリンクを作成します。(スクリプトがない場合は手動で作成)
3. **Homebrewパッケージのインストール:**
    `Brewfile`に記載されたパッケージをインストールします。

    ```bash
    brew bundle install --file ~/dotfiles/Brewfile
    ```

4. **シェルの設定:**
    デフォルトシェルをFishに変更します。
5. **各ツールの設定:**
    必要に応じて各ツールの設定を読み込みます (例: `anyenv`の初期化)。

## ツールの設定詳細

### Shell: Fish (`.config/fish/`)

- **設定ファイル:** `.config/fish/config.fish`
- **プロンプト:** Tide promptを使用。
- **エイリアス:**
  - `g`: `git`
  - `vim`/`vi`: `nvim`
  - `ws`: `open -a webstorm` (WebStormを開く)
  - `ll`, `la`, `lt`: `eza` (モダンな`ls`) を使ったファイル一覧表示
- **パス設定:** `anyenv`, `rbenv`, `Homebrew`などのパスが設定されています。
- **プラグイン:** `fish_plugins`ファイルで管理 (詳細はファイル参照)。

### Editor: Neovim (`.config/nvim/`)

- **設定:** LazyVimをベースにしており、設定はLuaで記述されています。
- **起動ファイル:** `init.lua`
- **プラグイン管理:** `config/lazy.lua` にてLazyVimをセットアップ。
- **プラグイン設定:** `plugins/`ディレクトリ以下に各プラグインごとの設定ファイルが配置されています。
  - `copilot-chat.lua`: GitHub Copilot Chat
  - `neo-tree.lua`: ファイルエクスプローラー
  - `telescope.lua`: ファジーファインダー
  - `lsp.lua`: LSP (Language Server Protocol) の設定

### Terminal Multiplexer: Tmux (`.config/tmux/`)

- **設定ファイル:** `.tmux.conf`
- **ベース:** [gpakosz/.tmux](https://github.com/gpakosz/.tmux) の設定をベースにしています。
- **カスタム設定:** `.tmux.conf.local` に個別の設定を記述します。
- **主要なキーバインド (Prefix: `C-b`):**
  - `r`: 設定のリロード
  - `|`: ペインを垂直に分割
  - `-`: ペインを水平に分割
  - `h, j, k, l`: Vimライクなペイン移動
  - `+`: 現在のペインを最大化

### Git & Lazygit

#### Git (`.gitconfig`)

- **エディタ:** `nvim`
- **エイリアス:**
  - `st`: `status`
  - `co`: `checkout`
  - `cm`: `commit`
  - `hist`: 整形されたログ表示
  - `open`: GitHubリポジトリをブラウザで開く
- **Commitizen (`.czrc`):**
  - `cz-emoji`を利用して、絵文字付きのConventional Commitsをサポートします。
  - 例: `:sparkles: feat: 新機能を追加`

#### Lazygit (`lazygit/config.yml`)

- Nerd Fontsのアイコン表示を有効化。
- **カスタムキーバインド:**
  - `C`: `git cz` を実行し、Commitizenを起動します。

### パッケージ管理: Homebrew (`Brewfile`)

`Brewfile`により、以下のツールやアプリケーションが管理されています。

- **CLIツール:** `fish`, `git`, `lazygit`, `neovim`, `tmux`, `anyenv`, `eza`, `peco`, `ghq`, `tree-sitter`, `ripgrep`, `fd` など。
- **GUIアプリケーション:** `google-chrome`, `slack`, `iterm2`, `webstorm`, `visual-studio-code`, `docker`, `karabiner-elements` など。
- **フォント:** `font-hack-nerd-font`

### カスタム設定

- **Karabiner (`.config/karabiner/karabiner.json`):**
  macOSのキーボードをカスタマイズするための設定です。詳細は`karabiner.json`を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bear29ers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
