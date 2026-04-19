## fresh-keeper

> このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

# CLAUDE.md

このファイルは、Claude Code (claude.ai/code) がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

**Fresh Keeper** (expiry_tracker) は、食品の賞味期限を追跡するためのフルスタックWebアプリケーションです。ユーザーが食品の在庫を管理し、期限切れ前に通知を受け取ることができます。

### 構成
- **バックエンド**: Go (Echo + GORM + PostgreSQL)
- **フロントエンド**: React + TypeScript + Vite
- **データベース**: PostgreSQL
- **コンテナ化**: Docker + Docker Compose

### 開発履歴
- バックエンドAPI: 手動開発
- **フロントエンド**: Claude Code (claude.ai/code) によって完全に設計・実装

## アーキテクチャ

### バックエンド（Go）
コードベースは関心事の分離を明確にしたクリーンアーキテクチャパターンに従っています：

```
controller/ → router/ → usecase/ → repository/ → model/
                    ↓
                validator/
```

**主要なアーキテクチャレイヤー:**
- **Controller**: ユーザーと製品エンドポイントのHTTPハンドラー
- **Router**: ミドルウェア（CORS、CSRF、JWT認証）を含むHTTPルーティング
- **Usecase**: リポジトリとバリデーターの呼び出しを統合するビジネスロジック層
- **Repository**: GORMとPostgreSQLを使用するデータアクセス層
- **Model**: GORMアノテーション付きドメインエンティティ（User、Product）
- **Validator**: ozzo-validationを使用した入力検証

### フロントエンド（React + TypeScript）
**🤖 Claude Code によって設計・実装された現代的なReactアーキテクチャ:**

```
pages/ → hooks/ → services/ → API (バックエンド)
  ↓       ↓
components/ stores/
```

**主要なアーキテクチャレイヤー:**
- **Pages**: ページ全体のコンポーネント（ダッシュボード、製品管理など）
- **Components**: 再利用可能なUIコンポーネント（フォーム、リスト、レイアウト）
- **Hooks**: React Queryとカスタムロジックによるデータフェッチング
- **Services**: APIクライアントとバックエンド通信サービス
- **Stores**: Zustandによるグローバル状態管理（認証、UI状態）
- **Types**: TypeScript型定義による型安全性の確保

**フロントエンド技術スタック:**
- **React 18**: 関数コンポーネント + Hooks
- **TypeScript**: 型安全性とコード品質
- **Vite**: 高速な開発環境
- **Material-UI**: 日本語対応UIコンポーネント
- **React Router**: SPA ルーティング
- **React Query**: 効率的なデータフェッチング・キャッシュ
- **Zustand**: シンプルな状態管理
- **React Hook Form + Zod**: 型安全なフォーム管理

### セキュリティ実装
- 保護されたエンドポイントのJWTベース認証
- トークンとクッキーを使用したCSRF保護
- フロントエンド統合のためのCORS設定
- セキュアなクッキー設定（HTTPOnly、SameSite）
- TypeScriptによる型安全なAPI通信

## 開発コマンド

### ローカル開発
```bash
# アプリケーションをローカルで実行
go run main.go

# アプリケーションをビルド
go build -o main .

# Docker Composeで実行（開発環境推奨）
docker-compose up -d

# Dockerサービスを停止
docker-compose down
```

### データベース
```bash
# データベースはDockerでPostgreSQLを使用
# 接続詳細はdocker-compose.ymlに記載:
# - Host: localhost:5434 (外部)、db:5432 (内部)
# - Database: expiry_tracker
# - User: postgres
# - Password: postgres
```

### 環境変数
必要な環境変数（docker-compose.yml参照）：
- `POSTGRES_HOST`、`POSTGRES_PORT`、`POSTGRES_USER`、`POSTGRES_PW`、`POSTGRES_DB`
- `SECRET`: JWT署名キー
- `API_DOMAIN`: CSRFクッキーのドメイン
- `FE_URL`: CORS用フロントエンドURL
- `PORT`: アプリケーションポート（デフォルト: 8080）

## データモデル

### User（ユーザー）
- メール/パスワード認証
- JWTトークンベースセッション
- 論理削除サポート

### Product（製品）
- Userに属する（外部キー関係）
- 2つの賞味期限タイプ：`best_before`（賞味期限）と`use_by`（消費期限）
- `IsNotified`フラグでの通知追跡
- 数量管理
- 論理削除サポート

## APIエンドポイント

### パブリックエンドポイント
- `POST /signup` - ユーザー登録
- `POST /login` - ユーザーログイン
- `POST /logout` - ユーザーログアウト
- `GET /csrf` - CSRFトークン取得

### 保護されたエンドポイント（JWT必須）
- `GET /products` - ユーザーの製品一覧
- `POST /products` - 製品作成
- `GET /products/:id` - 製品詳細取得
- `PUT /products/:id` - 製品更新
- `DELETE /products/:id` - 製品削除

## 主要な依存関係

- **Echo v4**: Webフレームワーク
- **GORM**: PostgreSQL用ORM
- **JWT**: 認証
- **ozzo-validation**: 入力検証
- **PostgreSQL**: データベース
- **Docker**: コンテナ化

## 開発上の注意事項

- アプリケーションはmain.goで依存性注入パターンを使用
- すべてのデータベース操作はリポジトリ層を通過
- 入力検証は専用のバリデーター構造体で処理
- コードベースはテスト支援のため主要コンポーネントにインターフェースを使用
- マイグレーションファイルは`migrations/`ディレクトリに配置

## 日本語での対応

このプロジェクトは日本語での対応を想定しています。Claude Codeとやりとりする際は日本語で回答してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yamagic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
