## ham-media-app

> ハムメディアは国内初のハムスター専門支援団体が運営するアプリケーションです。このプロジェクトは以下の主要機能を提供しています：

# ハムメディアアプリ - Claude Code ガイド

## プロジェクト概要

ハムメディアは国内初のハムスター専門支援団体が運営するアプリケーションです。このプロジェクトは以下の主要機能を提供しています：

1. **病院検索機能** - ハムスター受付可能病院の検索
2. **LINE Bot** - LINE上での病院検索と里親募集情報の提供
3. **管理画面** - 病院管理、在庫管理、スタッフ管理

## 技術スタック

### バックエンド

- **フレームワーク**: Fastify
- **データベース**: PostgreSQL
- **ORM**: Prisma
- **GraphQL**: Mercurius (Fastify用GraphQLプラグイン)
- **認証**: JWT
- **テスト**: Vitest + Mercurius Integration Testing
- **デプロイ**: Heroku

### フロントエンド

- **フレームワーク**: Next.js 15 (App Router)
- **スタイリング**: Panda CSS
- **GraphQL**: Apollo Client
- **UI コンポーネント**: Ark UI
- **認証**: Cookie ベース認証
- **デプロイ**: CloudFront/S3/Lambda@Edge

### 共通ツール

- **モノレポ管理**: pnpm workspace
- **Linting**: ESLint
- **フォーマット**: Prettier
- **CI/CD**: GitHub Actions
- **モニタリング**: Sentry

## プロジェクト構造

```
ham-media-app/
├── backend/           # バックエンドAPIサーバー
│   ├── server/        # Fastifyアプリケーション
│   │   ├── graphql/   # GraphQLスキーマとリゾルバー
│   │   ├── services/  # ビジネスロジック
│   │   └── routes/    # RESTエンドポイント
│   ├── prisma/        # データベーススキーマとシード
│   └── tests/         # バックエンドテスト
├── frontend/          # Next.jsフロントエンド
│   ├── app/           # App Router ページ
│   ├── components/    # UIコンポーネント
│   └── services/      # APIクライアント
└── graphql/           # 共有GraphQLスキーマ
```

## セットアップ手順

### 前提条件

- Node.js >= 22.16.0
- pnpm >= 10.14.0
- PostgreSQL データベース

### 初期セットアップ

1. **リポジトリクローン**

```bash
git clone git@github.com:egurinko/ham-media-app.git
cd ham-media-app
```

2. **依存関係インストール**

```bash
pnpm install
```

### バックエンドセットアップ

1. **環境変数設定**

```bash
cd backend
cp .env.template .env
# .envファイルを編集してDATABASE_URLなどを設定
```

2. **データベースセットアップ**

```bash
# スキーマをデータベースに反映
pnpm db:push

# Prismaクライアント生成
pnpm db:reflect

# 開発用シードデータ投入
pnpm db:seed:dev
```

3. **開発サーバー起動**

```bash
pnpm dev  # ポート3000で起動
```

### フロントエンドセットアップ

1. **環境変数設定**

```bash
cd frontend
cp .env.template .env
# .envファイルを編集
```

2. **開発サーバー起動**

```bash
pnpm dev  # ポート8080で起動
```

## 開発コマンド

### バックエンド

```bash
# 開発サーバー起動
pnpm backend dev

# ビルド
pnpm backend build

# テスト実行
pnpm backend test

# テスト（カバレッジ付き）
pnpm backend test:ci

# データベース操作
pnpm backend db:push          # スキーマをDBに反映
pnpm backend db:reflect       # Prismaクライアント生成
pnpm backend db:seed:dev      # 開発用シード実行

# コード品質
pnpm backend lint             # ESLint実行
pnpm backend format:check     # Prettier チェック
pnpm backend tscheck          # TypeScript型チェック
```

### フロントエンド

```bash
# 開発サーバー起動
pnpm frontend dev

# ビルド
pnpm frontend build

# GraphQL コード生成
pnpm frontend generate

# スタイル生成
pnpm frontend prepare

# コード品質
pnpm frontend lint            # Next.js lint実行
pnpm frontend format:check    # Prettier チェック
pnpm frontend tscheck         # TypeScript型チェック
```

## データベース

### スキーマ概要

- **Hospital**: 病院情報
- **HospitalAddress**: 病院住所・地理情報
- **Product**: 商品情報（在庫管理用）
- **Stock**: 在庫情報
- **StockRequest**: 在庫リクエスト
- **InternalUser**: 内部ユーザー（スタッフ）

### 主要な関係性

- 病院は住所情報を持ち、地理座標で位置検索が可能
- 商品は複数の在庫を持ち、在庫リクエストで管理
- 内部ユーザーはロールベースの認可システム

## API構成

### GraphQL API

- **Public API**: 一般ユーザー向け（病院検索など）
- **Internal API**: 管理画面向け（認証必須）

### REST API

- **Webhook**: LINE Bot用エンドポイント
- **Health Check**: ヘルスチェック用エンドポイント

## テスト

### バックエンドテスト

- **フレームワーク**: Vitest
- **統合テスト**: Mercurius Integration Testing
- **テストDB**: 専用のテスト環境設定

### テスト実行方法

```bash
# バックエンドテスト
cd backend
pnpm test

# カバレッジ付きテスト
pnpm test:ci
```

## デプロイ

### 本番環境

- **バックエンド**: Heroku
- **フロントエンド**: AWS (CloudFront + S3 + Lambda@Edge)

### CI/CD

GitHub Actionsを使用した自動デプロイメント

## 開発ガイドライン

### コード品質

- ESLint + Prettier による自動フォーマット
- TypeScript strict モード
- 全てのAPIエンドポイントにテストを記述

### Git ワークフロー

- main ブランチで開発
- Pull Request でのコードレビュー
- Renovate による依存関係自動更新

### 注意事項

- 本番環境では環境変数でシークレット管理
- データベース変更時はmigrationファイル作成
- GraphQLスキーマ変更時は影響範囲を確認

## 外部サービス連携

### LINE API

- LINE Bot SDK を使用
- Webhook 処理でメッセージ対応

### Google Maps API

- 病院位置情報の表示
- 地理座標による検索

### Sentry

- エラー監視・ログ収集
- パフォーマンス監視

## トラブルシューティング

### よくある問題

1. **データベース接続エラー**
   - `.env` ファイルの `DATABASE_URL` を確認
   - PostgreSQL サーバーが起動しているか確認

2. **GraphQL コード生成エラー**
   - バックエンドサーバーが起動しているか確認
   - スキーマファイルが最新か確認

3. **ポート競合**
   - バックエンド: 3000 番ポート
   - フロントエンド: 8080 番ポート
   - 他のプロセスが使用していないか確認

### ログ確認

```bash
# バックエンドログ
pnpm backend dev

# フロントエンドログ
pnpm frontend dev
```

## 追加リソース

- [プロジェクトREADME](./README.md)
- [ハムメディア公式サイト](https://ham-media.net)
- [病院検索アプリ](https://ham-media-app.net/hospitals)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/egurinko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/egurinko)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
