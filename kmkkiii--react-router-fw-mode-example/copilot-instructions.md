## react-router-fw-mode-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 開発コマンド

```bash
# 開発サーバー (HMR付き、http://localhost:5173)
npm run dev

# 本番ビルド
npm run build

# 本番サーバー起動
npm start

# 型チェック
npm run typecheck

# Lint とフォーマット (Biome)
npm run lint           # 問題をチェック
npm run lint:fix       # 自動修正
npm run format         # コードをフォーマット

# テスト (Vitest)
npm test               # 全テストを一度実行
npm run test:watch     # Watch モード

# データベース (Drizzle + PostgreSQL)
npm run db:push        # スキーマ変更をデータベースに反映
```

### 単一テストファイルの実行

```bash
npx vitest app/path/to/file.test.ts
```

## アーキテクチャ概要

### 技術スタック

- **フレームワーク**: React Router v7 + SSR (サーバーサイドレンダリング)
- **認証**: better-auth (メール/パスワード)
- **データベース**: PostgreSQL + Drizzle ORM
- **フォームバリデーション**: Conform + Zod (サーバー/クライアント共有スキーマ)
- **スタイリング**: Tailwind CSS v4
- **Lint/フォーマット**: Biome (ESLint + Prettier の代替)
- **テスト**: Vitest + React Testing Library

### コード構成

このプロジェクトは **Feature-based アーキテクチャ** に従っています：

```
app/
├── entities/           # ドメインエンティティとデータアクセス層
│   └── todo/
│       ├── repository.server.ts    # データベース操作 (CRUD)
│       └── types.ts                # 型定義
├── features/           # 機能モジュール（UI とロジック）
│   ├── auth/
│   │   ├── components/            # ログイン/サインアップフォーム
│   │   └── schemas/               # バリデーションスキーマ (Zod)
│   └── todo/
│       ├── components/            # Todo UI コンポーネント
│       └── schemas/               # バリデーションスキーマ (Zod)
├── shared/             # 共有インフラ
│   ├── auth/
│   │   ├── auth.server.ts         # サーバーサイド認証インスタンス
│   │   └── auth.client.ts         # クライアントサイド認証クライアント
│   └── db/
│       └── client.server.ts       # データベースクライアント
├── routes/             # React Router ファイルベースルート
│   ├── _index.tsx                 # ランディングページ
│   ├── login.tsx                  # ログインページ
│   ├── signup.tsx                 # サインアップページ
│   ├── todos.tsx                  # Todo アプリ（保護されたページ）
│   └── api.auth.$.tsx             # 認証 API エンドポイント
├── test/               # テストユーティリティ
│   ├── setup.ts                   # テスト設定 (jest-dom)
│   ├── helpers.tsx                # renderWithRouter()
│   └── factories.ts               # テストデータファクトリ
└── routes.ts           # ルート設定
```

### 主要なアーキテクチャパターン

**1. サーバー専用コード**: `.server.ts` で終わるファイルはサーバー専用で、クライアントバンドルから除外されます。データベースアクセス、機密操作、環境変数の使用に利用します。

**2. ルート設定**: `app/routes.ts` は `flatRoutes()` を使用してファイルベースルーティングを実現します。テストファイル（`**/*.test.ts(x)`）は `ignoredRouteFiles` により自動的に除外されます。

**3. 認証フロー**:
- better-auth がセッション管理を担当
- `auth.server.ts` がサーバーサイド認証インスタンスをエクスポート
- ルートは `loader` 関数内で認証状態をチェック
- 未認証の場合は `/login` にリダイレクト

**4. データベース層**:
- スキーマは `db/schema.ts` で Drizzle ORM を使用して定義
- `app/entities/*/repository.server.ts` で Repository パターンを採用
- Drizzle のクエリは型安全で、スキーマのエクスポートを使用

**5. テスト戦略**:
- コンポーネントテストには `app/test/helpers.tsx` の `renderWithRouter()` を使用
- テストデータには `app/test/factories.ts` の `buildTodo()` ファクトリを使用
- サーバーサイドモジュール（db, auth）は `vi.mock()` でモック化
- 全てのテストファイルは `*.test.ts` または `*.test.tsx` 拡張子が必須

**6. フォームバリデーション**:
- Conform + Zod でサーバー/クライアント両方でバリデーションを実行
- スキーマは `app/features/*/schemas/` に配置
- `useForm` で onBlur/onInput バリデーションを実装
- `parseWithZod` は `@conform-to/zod/v4` から import（Zod v4 対応）
- action は `{ lastResult: submission.reply() }` を返す
- フォームコンポーネントは `lastResult` を props で受け取る

## 重要な注意事項

- **パスエイリアス**: `~/` は `app/` にマッピングされます（tsconfig.json で設定）
- **データベース**: PostgreSQL の実行が必要です（ローカル開発には `docker compose up` を使用）
- **環境変数**: `.env.example` を `.env` にコピーして `DATABASE_URL` を設定してください
- **テストファイル**: `.test.ts(x)` 拡張子を使用しないとルートとして扱われてしまいます
- **Conform + Zod v4**: `@conform-to/zod/v4` から import することで Zod v4 を使用可能

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmkkiii) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
