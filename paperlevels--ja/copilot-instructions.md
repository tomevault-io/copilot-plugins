## ja

> - Cloudflare D1（SQLite）

# Paperlevels

ログラインで需要を測るPoCサイト。

## Tech Stack

- Astro 6
- TypeScript
- Tailwind CSS v4
- shadcn/ui
- Cloudflare D1（SQLite）
- Cloudflare Workers（デプロイ先）

## Getting Started

### 1. 依存関係をインストール

```bash
npm install
```

### 2. D1 スキーマを適用

```bash
# ローカル
npm run db:local

# リモート（本番）
npm run db:remote
```

`wrangler.toml` に D1 バインディング（`DB` / `paperlevels-db`）が設定されている必要があります。

### 3. 開発サーバーを起動

```bash
npm run dev
```

ブラウザで [http://localhost:4321](http://localhost:4321) を開きます。

> D1 は Workers バインディング経由のため、Supabase 用の `.env.local` は不要です。

## テストの実行

このプロジェクトでは、Vitest で API / DB / データ層のテストを、Playwright で E2E テストを実行します。

### Unit / Integration テスト

```bash
npm run test
```

UI で確認したい場合:

```bash
npm run test:ui
```

個別に実行する場合の例:

```bash
npm exec vitest run tests/api/loglines.test.ts
```

### E2E テスト

```bash
npm run test:e2e
```

特定ファイルだけ実行する場合の例:

```bash
npx playwright test e2e/post-logline.spec.ts
```

---

## Admin

管理画面（`/admin`）は **認証なし**（個人利用前提）です。

公開環境では Cloudflare Access や IP 制限などの保護を検討してください。

---

## Cloudflare へのデプロイ

Cloudflare Workers へのデプロイ方法、D1 接続の設定などは以下を参照してください。

👉 [README_Cloudflare.md](./README_Cloudflare.md)

👉 [README_sqlite.md](./README_sqlite.md)

---

## Features

- ログイン不要でログラインを投稿
- シェア数に基づく人気順 / 新着順のソート
- SNSシェア機能（Web Share API / X Intent）
- コメント機能（Markdown対応）
- コメント内URLのOGPプレビュー（microlink.io）
- 管理画面（削除・カテゴリ編集）

---
> Source: [paperlevels/ja](https://github.com/paperlevels/ja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
