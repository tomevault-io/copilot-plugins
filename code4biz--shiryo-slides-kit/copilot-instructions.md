## shiryo-slides-kit

> このファイルは、**Claude Code / Codex などの AI コーディングエージェント**が、

# AGENTS.md — AI エージェント向けセットアップ指示書

このファイルは、**Claude Code / Codex などの AI コーディングエージェント**が、
このリポジトリ（資料作成キット shiryo-slides-kit）をユーザーの環境に導入するための手順書です。
ユーザーから「このリポジトリを入れて」「この URL を読んでセットアップして」と頼まれたら、この手順に従ってください。

人間の読者向けの説明は `README.md` にあります。このファイルはエージェントの作業手順に絞っています。

---

## ゴール（この 3 つが揃ったら完了）

1. スキル一式が、ユーザーが使うツールの**スキル置き場**に置かれている（下表。SKILL.md の形式は共通なので中身は同一でよい）

   | ツール | Mac / Linux | Windows |
   |---|---|---|
   | Claude Code | `~/.claude/skills/shiryo-slides/` | `%USERPROFILE%\.claude\skills\shiryo-slides\` |
   | Codex CLI | `~/.codex/skills/shiryo-slides/` | `%USERPROFILE%\.codex\skills\shiryo-slides\` |

2. **Node.js と Playwright（chromium 含む）** が動く
3. 動作確認が済んでいて、ユーザーがそのまま「この議事録から提案資料を作って」と頼める状態

**どちらに入れるか**: 原則、**いまこのセットアップを実行しているエージェント自身のツール**に入れる（あなたが Claude Code なら `.claude`、Codex なら `.codex`）。ユーザーが両方使っている・別のツールに入れたいと言った場合は、その指定に従う（両方に入れても問題ない）。

## 進め方の原則

- **環境は自分で調べる。** ユーザーへの質問は最小限にする
- ただし **OS 全体に影響するインストール**（Node.js 本体・git 本体など）は、実行前に何を入れるか一言伝えてから進める
- 失敗したら同じコマンドを繰り返さず、下のフォールバック表で**別の手段に切り替える**
- **スキルのファイルは書き換えない。** そのままコピーする。「改善」もしない
- ユーザーの既存のスキル置き場（`~/.claude/skills/` / `~/.codex/skills/`）にある**他のスキルには触れない**

---

## 手順

### 1. 環境を調べる

以下を確認する（無いものがあっても、この時点ではエラーにしない）。

- OS（Windows / Mac / Linux）とシェル
- `git --version` / `node -v` / `npm -v` が通るか
- 入れる先のスキル置き場（ゴールの表参照）があるか（無ければ後の手順で作る）

### 2. リポジトリを取得する

**git がある場合**（一時的な作業ディレクトリで）:

```bash
git clone https://github.com/code4biz/shiryo-slides-kit.git
```

**git が無い場合** — git を新規インストールする必要はありません。ZIP で足ります:

```bash
# Mac / Linux
curl -L -o shiryo-slides-kit.zip https://github.com/code4biz/shiryo-slides-kit/archive/refs/heads/main.zip
unzip shiryo-slides-kit.zip
```

```powershell
# Windows（PowerShell）
Invoke-WebRequest -Uri "https://github.com/code4biz/shiryo-slides-kit/archive/refs/heads/main.zip" -OutFile shiryo-slides-kit.zip
Expand-Archive shiryo-slides-kit.zip -DestinationPath .
```

（ZIP 展開後のフォルダ名は `shiryo-slides-kit-main` になります）

### 3. スキルを配置する

`skills/shiryo-slides` フォルダを**丸ごと**、ゴールの表で決めた置き場にコピーする。以下は Claude Code（`.claude`）の例。**Codex に入れる場合は `.claude` を `.codex` に読み替える**（それ以外は完全に同じ）。

```bash
# Mac / Linux
mkdir -p ~/.claude/skills
cp -R <取得したフォルダ>/skills/shiryo-slides ~/.claude/skills/
```

```powershell
# Windows（PowerShell）
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse -Force "<取得したフォルダ>\skills\shiryo-slides" "$env:USERPROFILE\.claude\skills\"
```

- **既に `shiryo-slides` フォルダがある場合**は、上書きしてよいかユーザーに確認してから置き換える
- 置くのは `skills/shiryo-slides` **だけ**。リポジトリ全体や `showcase/` を skills 配下に置かない

### 4. Node.js / Playwright を整える

1. `node -v` が通らない場合 — 何を入れるか伝えたうえで:
   - Mac: `brew install node`（Homebrew が無ければ https://nodejs.org の LTS 版インストールを案内）
   - Windows: `winget install OpenJS.NodeJS.LTS`
   - インストール後は**新しいシェルで** `node -v` を確認する（PATH の反映に開き直しが必要）
2. Playwright を入れる:
   ```bash
   npm install -g playwright
   npx playwright install chromium
   ```
   - `npx playwright install chromium` は**約 400MB のダウンロード**が走る。時間がかかることを先にユーザーへ伝える
   - `npm -g` が権限エラーになったら、ユーザーの作業フォルダ内で `npm install playwright` にフォールバックする（スキルはローカル/グローバル両方を探すので、どちらでも動く）

### 5. 動作確認

```bash
# Mac / Linux（Codex の場合は .claude を .codex に読み替え）
ls ~/.claude/skills/shiryo-slides/SKILL.md
NODE_PATH="$(npm root -g)" node -e "require('playwright'); console.log('OK')"
```

```powershell
# Windows（PowerShell）
Test-Path "$env:USERPROFILE\.claude\skills\shiryo-slides\SKILL.md"
$env:NODE_PATH = npm root -g
node -e "require('playwright'); console.log('OK')"
```

両方通れば導入完了。**Claude Code / Codex とも、再起動しないと新しいスキルを読み込まない**ので、その旨をユーザーに伝える。

### 6. 完了報告

次を短く報告する。

- スキルを置いた場所（どのツール向けか）
- 新しくインストールしたもの / 既に入っていてスキップしたもの
- ツール（Claude Code / Codex）の再起動が必要なこと
- 最初の一歩の例: 「この議事録から提案資料を作って」「勉強会向けに図解多めのスライドにして」（Codex では `$` プレフィックスや `/skills` からの明示呼び出しも可能）

---

## トラブル時のフォールバック

| 症状 | 対処 |
|---|---|
| git が無い | ZIP ダウンロード（手順 2）。git のインストールは不要 |
| `npm -g` が権限エラー | ユーザーの作業フォルダ内にローカルインストール。`sudo` は勧めない |
| プロキシで chromium がダウンロードできない | その旨を報告する。Playwright 無しでもスライド生成自体は可能（視覚検証がスキップされる）と伝える |
| Google Fonts が届かない環境 | `README.md` §7「フォントの警告が出る」をユーザーに案内する |
| `node -v` がインストール直後に通らない | シェルを開き直す（PATH 未反映が原因のことが多い） |

## やらないこと

- スキル本体（`skills/shiryo-slides/` 配下）の書き換え・「改善」・翻訳
- ユーザーの他のスキルの削除・変更
- リポジトリ全体をスキル置き場に置くこと（置くのは `skills/shiryo-slides` だけ）
- Playwright インストールの失敗を理由にセットアップ全体を中断すること（スキル配置まで済ませて、残作業を報告する）

---
> Source: [code4biz/shiryo-slides-kit](https://github.com/code4biz/shiryo-slides-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
