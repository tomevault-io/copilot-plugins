## mtga-deck-builder

> Magic: The Gathering ARENA 向けの **ブラウザ上で動作するデッキビルダーアプリ**を開発します。ユーザーはカード検索、フィルター、デッキ保存、共有、エクスポート（Arena用インポート形式）などを行えるようにします。

# CLAUDE.md

## 📘 プロジェクト概要

Magic: The Gathering ARENA 向けの **ブラウザ上で動作するデッキビルダーアプリ**を開発します。ユーザーはカード検索、フィルター、デッキ保存、共有、エクスポート（Arena用インポート形式）などを行えるようにします。

このファイルは Claude Code に対して、仕様や制約・目的を明示するためのガイドラインです。

---

## ✅ タスクの目的（GOAL）

- モダンな UI を持つ Web デッキビルダーを作成する。
- Magic: The Gathering ARENA にインポート可能な形式でデッキをエクスポートできるようにする。
- MTG のカード情報（例：Scryfall API）を取得して検索・フィルタリングできるようにする。

---
## 🔍 仕様・要件

仕様書は：docs/requirements.md を参照してください。

---

## 🤖 Claude へのお願い（INSTRUCTIONS）

- 日本語で応答して下さい。
- Todoを一つ消化する度に必ず、git commitしてください。コミットメッセージも日本語で記述してください。 
また、1機能1コミットで小さくコミットして下さい。

テスト駆動開発（TDD）を採用し、以下のステップで実装を進めてください：

1. Red - まず失敗するテストを書き、git commitする。
2. Green - テストを通すための最小限のコードを書き、git commitする。
3. Refactor - コードをリファクタリングして改善し、git commitする。

Git Commit の際は、以下のルールに従ってください：

- Conventional Commits のルールに従ってコミットメッセージを記述してください。
- テストを書いたら: test: [機能] の失敗するテストを追加
- テストを通したら: feat: テストを通すために [機能] を実装
- リファクタリングしたら: refactor: [説明]

---

## 🔧 開発コマンド

### 基本コマンド
- `npm run dev` - 開発サーバー起動（Vite + HonoX）
- `npm run build` - 本番ビルド（クライアント + サーバー）
- `npm run preview` - Wranglerでローカルプレビュー
- `npm run deploy` - Cloudflare Workers にデプロイ

### コード品質
- `npm run lint` - ESLint実行
- `npm run lint:fix` - ESLint自動修正
- `npm run format` - Prettier実行

### 重要事項
- TypeScriptエラーとESLintエラーはコミット前に必ず解消すること
- Husky + lint-stagedが設定済み（コミット時に自動でlint/format実行）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/torounit)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/torounit)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
