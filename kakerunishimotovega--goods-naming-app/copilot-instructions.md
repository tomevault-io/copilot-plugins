## goods-naming-app

> このプロジェクトは、従来GASとスプレッドシートで構築されていた商品命名アプリを、より堅牢な構成に置き換えるものです。

# LOWYA商品命名アプリ開発ガイド

## プロジェクト概要
このプロジェクトは、従来GASとスプレッドシートで構築されていた商品命名アプリを、より堅牢な構成に置き換えるものです。

## 技術スタック
- **バックエンド**: Google Apps Script (TypeScript)
- **データベース**: Supabase (PostgreSQL)
- **スタイリング**: Tailwind CSS（導入予定）
- **ビルドツール**: esbuild
- **デプロイツール**: clasp

## 重要な制約とルール

### 1. GAS関数の公開方法（最重要）
バックエンド（`src/*.ts`）で定義した関数をGASから呼び出すには、必ず以下の記述が必要です：

```typescript
// 例: src/index.ts
function myFunction() {
  // 処理
}

// GASのグローバルスコープに公開
(global as any).myFunction = myFunction;
```

この記述により、`build.js`を通して`dist/main.js`に正しく出力され、GASから呼び出し可能になります。

### 2. ビルドとデプロイコマンド
```bash
# ビルド（TypeScriptをバンドル）
npm run build

# デプロイ（ビルド + clasp push）
npm run deploy
```

### 3. データベース通信
- Supabase (PostgreSQL) を使用
- **開発環境（Claude Code）**: Supabase MCPツールを使用してデータベース操作を行う
  - テーブル確認: `mcp__supabase__list_tables`
  - SQL実行: `mcp__supabase__execute_sql`
  - マイグレーション: `mcp__supabase__apply_migration`
  - 型生成: `mcp__supabase__generate_typescript_types`
- **GAS環境**: `UrlFetchApp`でSupabase REST APIを直接呼び出す
  - `@supabase/supabase-js`はGAS上では動作しない
  - 環境変数はスクリプトプロパティから取得（`PropertiesService.getScriptProperties()`）

### 4. ファイル構成
```
src/
  ├── index.ts          # エントリーポイント
  ├── index.html        # フロントエンド
  ├── appsscript.json   # GAS設定ファイル
  └── *.ts              # その他のTypeScriptファイル
dist/
  ├── main.js           # ビルド後のJavaScript
  ├── index.html        # コピーされたHTML
  └── appsscript.json   # コピーされた設定ファイル
build.js                # カスタムビルドスクリプト
```

### 5. コーディング規約
- TypeScriptで記述
- GASから呼び出す全ての関数は`(global as any).関数名 = 関数名;`で公開
- フロントエンドのスタイリングはTailwind CSSを使用（導入予定）
- テストはvitestで記述

## ビルドプロセスの仕組み
1. `build.js`が`src/index.ts`をesbuildでバンドル
2. `dist/main.js`に出力
3. コードの先頭に`var global = this;`を追加
4. `global.関数名 =`のパターンを検出
5. ファイル末尾にGAS用のラッパー関数を自動生成
6. `appsscript.json`と`index.html`を`dist/`にコピー

## 注意事項
- GAS環境ではNode.jsのモジュールやブラウザAPIの一部が使えない
- 外部API通信は`UrlFetchApp`を使用
- 非同期処理はGAS独自の制約がある
- ビルド前に必ず型チェックとテストを実行

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KakeruNishimotoVega)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KakeruNishimotoVega)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
