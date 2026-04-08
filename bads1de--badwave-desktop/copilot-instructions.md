## badwave-desktop

> Electron、Next.js、TypeScript で構築された、ローカルファーストなデスクトップ音楽プレイヤーアプリケーションです。Spotify や Apple Music のようなアーキテクチャを採用し、UI のパフォーマンス向上とオフライン対応のためにローカルデータベース（SQLite）からの読み込みを優先し、バックグラウンドで Supabase と同期を行います。

# BadWave (Desktop)

Electron、Next.js、TypeScript で構築された、ローカルファーストなデスクトップ音楽プレイヤーアプリケーションです。Spotify や Apple Music のようなアーキテクチャを採用し、UI のパフォーマンス向上とオフライン対応のためにローカルデータベース（SQLite）からの読み込みを優先し、バックグラウンドで Supabase と同期を行います。

## プロジェクト概要

- **タイプ:** デスクトップアプリケーション (Electron + Next.js)
- **コア・フィロソフィー:** **ローカルファースト (Local-First)**。UI は主にローカルの SQLite データベースからデータを読み込み、ゼロレイテンシの操作感とオフライン対応を実現します。データはバックグラウンドで Supabase から同期されます。
- **オペレーティングシステム:** Windows（フォルダ名に基づく主要ターゲット）ですが、クロスプラットフォームを考慮した構造になっています。

## アーキテクチャ

### 技術スタック

- **フロントエンド:** Next.js (React), Tailwind CSS, Radix UI
- **状態管理:** Zustand, TanStack Query (React Query)
- **デスクトップシェル:** Electron
- **ローカルデータベース:** SQLite (`better-sqlite3` および `drizzle-orm` を使用)
- **リモートデータベース:** Supabase (PostgreSQL)
- **ORM:** Drizzle ORM

### データフロー (ローカルファースト)

1. **読み込み (UI):** コンポーネントは `useGet...` フックを使用して、Electron IPC 経由でローカル SQLite DB から _のみ_ データを読み込みます。これにより、UI が高速に動作し、オフラインでも利用可能になります。
2. **同期 (バックグラウンド):** 専用の `useSync...` フックが Supabase からデータを取得し、ローカル SQLite DB に「Upsert（更新または挿入）」します。
3. **反映:** 同期が成功すると、React Query のキャッシュキーが無効化され、ローカル DB からの再読み込みがトリガーされて UI が更新されます。

詳細な設計判断については `docs/LOCAL_FIRST_MIGRATION.md` を参照してください。

## ディレクトリ構成

### ファイル階層図

```text
badwave-windows/
├── app/              # Next.js App Router (レンダラープロセス)
│   ├── (site)/       # ルートページ
│   ├── account/      # アカウント設定
│   ├── liked/        #「いいね」した曲
│   ├── playlists/    # プレイリスト
│   ├── layout.tsx    # ルートレイアウト
│   └── globals.css   # グローバルスタイル
├── components/       # React UI コンポーネント
├── electron/         # Electron メインプロセス
│   ├── main.ts       # エントリポイント
│   ├── ipc/          # IPC ハンドラー
│   ├── db/           # ローカル SQLite 設定・スキーマ
│   └── preload/      # プリロードスクリプト
├── hooks/            # カスタム React フック
│   ├── data/         # 読み込み用フック (ローカル DB)
│   ├── sync/         # 同期用フック (Supabase -> ローカル DB)
│   ├── mutations/    # 変更用フック
│   └── stores/       # Zustand ストア
├── libs/             # ユーティリティ (Supabase クライアントなど)
├── drizzle/          # Drizzle マイグレーションファイル
├── providers/        # React Context プロバイダー
└── public/           # 静的アセット
```

### 各ディレクトリの役割

- `electron/`: Electron のメインプロセスコード
  - `main.ts`: エントリポイント
  - `ipc/`: IPC ハンドラー（フロントエンドとメインプロセスの通信）
  - `db/`: ローカル SQLite のスキーマとクライアント設定
- `app/`: Next.js App Router のページとレイアウト
- `hooks/`: React フック
  - `data/`: `useGet...` フック（ローカル DB からの読み込み用）
  - `sync/`: `useSync...` フック（リモートからローカルへの同期用）
- `components/`: React UI コンポーネント
- `drizzle/`: データベースのマイグレーションファイル

## 開発

### 主要コマンド

- **開発起動:** `npm run dev:electron` (Next.js と Electron を同時に起動)
- **インストーラー作成:** `npm run dist:electron`
- **テスト実行:** `npm test`
- **DB マイグレーション:** `drizzle-kit generate:sqlite` (`drizzle.config.ts` で管理)

### 開発コンベンション

- **TypeScript:** 厳格な型定義を推奨します。
- **IPC:** ネイティブ操作（DB アクセス、ファイルシステムなど）はすべて Electron IPC を経由する必要があります。
- **スタイリング:** すべてのスタイリングに Tailwind CSS を使用します。
- **テスト:** Jest を使用してユニットテストおよび統合テストが構成されています。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bads1de)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bads1de)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
