## lunch-map

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

ランチ店舗おすすめアプリケーション - ユーザーの現在地周辺にあるランチ店舗を表示し、レビュー機能付きのモバイルファーストWebアプリケーション

## アーキテクチャ

- **フロントエンド**: React + TypeScript + Vite (SPA)
- **バックエンド**: Hono + Cloudflare Workers
- **データベース**: Cloudflare D1 (SQLite)
- **ファイルストレージ**: Cloudflare R2
- **外部API**: Google Maps API、Google Places API
- **認証**: Google OAuth 2.0

## 開発環境セットアップ

### 必須の環境変数設定
`.env`ファイルを作成し、以下の値を設定:
```bash
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
JWT_SECRET=your-jwt-secret-key
GOOGLE_MAPS_API_KEY=your-google-maps-api-key
VITE_GOOGLE_MAPS_API_KEY=your-google-maps-api-key
```

### 開発用コマンド

```bash
# 依存関係のインストール
npm install

# データベースのマイグレーション（初回のみ）
npm run db:generate

# フロントエンド開発サーバー起動（ポート5173）
npm run frontend:dev

# バックエンド開発サーバー起動（ポート8787）
npm run dev

# 本番ビルド
npm run build

# 型チェック
npm run type-check

# Cloudflareにデプロイ
npm run deploy
```

## 主要ディレクトリ構成

```
src/
├── api/                # Hono バックエンドAPI
│   ├── index.ts       # メインエントリーポイント
│   └── routes/        # APIルート定義
├── frontend/          # React フロントエンド
│   ├── components/    # 再利用可能コンポーネント
│   ├── pages/         # ページコンポーネント
│   ├── hooks/         # カスタムフック（認証、位置情報）
│   └── utils/         # API呼び出し、ユーティリティ
├── types/             # TypeScript型定義
└── database/          # データベース関連
migrations/            # SQLマイグレーションファイル
```

## 重要な実装詳細

### 認証フロー
- Google OAuth 2.0を使用
- JWTトークンでセッション管理
- `/api/auth/google` でOAuth開始、`/api/auth/callback` でコールバック処理

### 位置情報とマップ
- `useLocation`フックで現在地取得
- Google Maps APIで地図表示と店舗マーカー
- Google Places APIで周辺店舗検索（半径1km）

### レビュー機能
- 1店舗につき1ユーザー1レビューの制限
- 星評価（1-5）、テキスト、画像（最大5枚、5MB以下）
- Cloudflare R2に画像アップロード

### データベーススキーマ
- `users`: ユーザー情報（Google OAuth連携）
- `reviews`: レビュー情報（評価、コメント）
- `review_images`: レビュー画像情報

## API エンドポイント

### 認証
- `GET /api/auth/google` - OAuth開始
- `GET /api/auth/callback` - OAuthコールバック
- `POST /api/auth/logout` - ログアウト

### ユーザー
- `GET /api/user/profile` - ユーザー情報取得
- `GET /api/user/reviews` - ユーザーのレビュー履歴

### 店舗
- `GET /api/places/nearby` - 周辺店舗検索
- `GET /api/places/details/:placeId` - 店舗詳細

### レビュー
- `GET /api/reviews/places/:placeId` - 店舗のレビュー取得
- `POST /api/reviews` - レビュー投稿
- `PUT /api/reviews/:id` - レビュー更新
- `DELETE /api/reviews/:id` - レビュー削除
- `POST /api/reviews/:id/images` - 画像アップロード

## 開発時の注意点

### Google APIs設定
- Google Cloud Consoleで以下のAPIを有効化:
  - Maps JavaScript API
  - Places API
  - OAuth 2.0
- 本番環境では適切なAPIキー制限を設定

### Cloudflare設定
- D1データベースの作成とバインディング設定
- R2バケットの作成とバインディング設定
- `wrangler.toml`で環境変数とバインディングを設定

### セキュリティ
- JWTシークレットは強力なランダム文字列を使用
- APIキーは環境変数で管理、フロントエンドには必要最小限のみ
- 画像アップロード時のファイル形式・サイズチェック

### モバイル対応
- レスポンシブデザイン（最大幅480px）
- タッチ操作最適化
- 位置情報許可が必要な旨をユーザーに説明

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hajimasa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hajimasa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
