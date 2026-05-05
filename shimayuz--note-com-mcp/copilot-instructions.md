## note-com-mcp

> ユーザーが「セットアップして」「設定して」「インストールして」「使えるようにして」と言ったら、以下の手順を実行してください。

# note MCP Server セットアップ指示書

## 重要: このファイルを最初に読むこと

ユーザーが「セットアップして」「設定して」「インストールして」「使えるようにして」と言ったら、以下の手順を実行してください。

---

## 🚀 推奨: セットアップスクリプトを使用

**最も確実な方法は、セットアップスクリプトを実行することです。**

### Macの場合
```bash
./scripts/setup.sh
```

### Windowsの場合
```powershell
.\scripts\setup.ps1
```

スクリプトが正常に完了したら、Cursorを再起動してください。

---

## 手動セットアップ（スクリプトが使えない場合）

以下の手順を**すべて順番に実行**してください。

### Step 1: Node.js確認

```bash
node --version
```

**v18未満またはエラーの場合は停止して以下を案内：**
- Mac: `brew install node`
- Windows: https://nodejs.org/ からダウンロード

**Windowsの場合は文字化け防止のため、最初に以下を実行：**
```powershell
# PowerShellでUTF-8を有効化
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$OutputEncoding = [System.Text.Encoding]::UTF8
chcp 65001
```

---

### Step 2: npmパッケージインストール

```bash
npm install
```

**エラー時：**
```bash
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

---

### Step 3: Playwrightインストール

```bash
npx playwright install
```

---

### Step 4: ビルド

```bash
npm run build
```

**確認：**
```bash
ls build/note-mcp-server.js
```

ファイルが存在すればOK。

---

### Step 5: 環境変数設定

```bash
# .env.sample から .env を生成
cp .env.sample .env
```

> **Note**: 認証情報の手動設定は不要です。
> Cursor再起動後、初回のMCPツール呼び出し時にブラウザが開き、ログインすると自動でセッションが取得されます。

---

### Step 6: プロジェクトパス取得

```bash
pwd
```

この出力を覚えておく（例: `/Users/username/note-com-mcp`）

---

### Step 7: OS判定

```bash
uname -s 2>/dev/null || echo "Windows"
```

- `Darwin` → Mac
- `Linux` → Linux  
- `Windows` または MINGW → Windows

---

### Step 8: MCP設定ファイル作成

#### Macの場合

```bash
mkdir -p ~/.cursor
```

以下の内容で `~/.cursor/mcp.json` を作成：

```json
{
  "mcpServers": {
    "note-api": {
      "command": "node",
      "args": ["ここにStep6のパス/build/note-mcp-server.js"]
    }
  }
}
```

**重要: `ここにStep6のパス` を実際のパスに置き換える**

#### Windowsの場合

```powershell
# ディレクトリ作成
New-Item -Path "$env:USERPROFILE\.cursor" -ItemType Directory -Force

# プロジェクトパス取得（Replace()メソッドを使用）
$projectPath = (Get-Location).Path.Replace('\', '\\')

# MCP設定ファイル作成（UTF-8で保存）
$config = @"
{
  "mcpServers": {
    "note-api": {
      "command": "node",
      "args": ["$projectPath\\build\\note-mcp-server.js"]
    }
  }
}
"@
$config | Out-File -FilePath "$env:USERPROFILE\.cursor\mcp.json" -Encoding utf8
```

**重要: 上記コマンドをプロジェクトディレクトリで実行すること**

> ⚠️ **エラーが発生した場合**: `docs/SETUP_ERRORS_AND_SOLUTIONS.md` を参照してください。

---

### Step 9: 完了報告

以下のように報告する：

```
◤◢◤◢◤◢◤◢◤◢◤◢◤◢
✅ セットアップ完了！

🚀 次のステップ:
1. Cursor を再起動
2. 「noteで記事を検索して」と試す
3. 初回はブラウザが開くので note.com にログイン
4. ログイン完了後、自動でセッションが取得されます
◤◢◤◢◤◢◤◢◤◢◤◢◤◢
```

---

## トラブルシューティング

### npm install失敗
```bash
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

### playwright install失敗
```bash
npx playwright install-deps
npx playwright install
```

### build失敗
```bash
rm -rf node_modules
npm install
npm run build
```

---

## コマンド早見表

| 目的 | コマンド |
|------|---------|
| インストール | `npm install` |
| ビルド | `npm run build` |
| 起動 | `npm run start` |
| HTTP起動 | `npm run start:http` |

---

## 注意事項

- 各Stepでエラーが出たら次に進まず、まずエラーを解決する
- MCP設定のパスは絶対パスで指定する
- Windowsのパスはバックスラッシュを `\\` でエスケープする
- セットアップ完了後は必ずCursorを再起動する

---
> Source: [shimayuz/note-com-mcp](https://github.com/shimayuz/note-com-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
