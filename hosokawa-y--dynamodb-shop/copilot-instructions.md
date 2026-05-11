## dynamodb-shop

> このファイルはClaude Codeがプロジェクトを理解するためのガイドです。

# CLAUDE.md

このファイルはClaude Codeがプロジェクトを理解するためのガイドです。

## 開発プラン

詳細な開発プランは以下のファイルを参照：
- `~/.claude/plans/quizzical-moseying-clover.md`
- `doc/実装計画.md`

## 担当範囲
ユーザーがユーザー担当のコードを実装する際に、コードの例と解説を表示してください。

### ユーザー担当（DynamoDB学習のため）

| カテゴリ | ファイル |
|---------|---------|
| ドメインモデル | `backend/internal/domain/*.go` |
| DynamoDBクライアント | `backend/internal/repository/dynamodb.go` |
| リポジトリ | `backend/internal/repository/*_repo.go` |
| サービス | `backend/internal/service/*.go` |

### Claude Code担当

| カテゴリ | ファイル |
|---------|---------|
| ミドルウェア | `backend/internal/middleware/*.go` |
| HTTPハンドラー | `backend/internal/handler/*.go` |
| ルーター設定 | `backend/internal/handler/router.go` |
| インフラスクリプト | `infrastructure/scripts/*.sh` |
| main.go接続 | `backend/cmd/api/main.go`（依存関係接続） |
| フロントエンド | `frontend/` |

## プロジェクト概要

DynamoDBの主要機能を学習するためのEコマースアプリケーション。

## 技術スタック

- **Backend**: Go 1.21+
  - フレームワーク: net/http (標準ライブラリ)
  - AWS SDK: aws-sdk-go-v2
  - 認証: JWT (golang-jwt/jwt)
- **Frontend**: Vue.js 3 + Vite
  - 状態管理: Pinia
  - HTTP: Axios
  - グラフ: Chart.js
- **Database**: AWS DynamoDB
  - Single Table Design
  - オンデマンドキャパシティ
- **Infrastructure**:
  - AWS Lambda (Go)
  - DynamoDB Streams

## ディレクトリ構成

```
dynamodb-shop/
├── backend/
│   ├── cmd/api/main.go           # エントリーポイント
│   ├── internal/
│   │   ├── config/               # 環境変数・設定
│   │   ├── domain/               # エンティティ・ドメインモデル
│   │   ├── repository/           # DynamoDB操作
│   │   ├── service/              # ビジネスロジック
│   │   ├── handler/              # HTTPハンドラー
│   │   └── middleware/           # 認証・CORS・ログ
│   └── pkg/response/             # レスポンスヘルパー
├── frontend/
│   └── src/
│       ├── views/                # ページコンポーネント
│       ├── components/           # 共通コンポーネント
│       ├── store/                # Pinia ストア
│       ├── api/                  # APIクライアント
│       └── router/               # Vue Router
├── lambda/
│   └── inventory-stream-handler/ # Streams処理Lambda
└── infrastructure/
    └── scripts/                  # AWS CLIスクリプト
```

## コーディング規約

### Go

- パッケージ名は小文字単数形
- エラーは必ずハンドリングする
- コンテキストは第一引数に渡す
- internal/ 以下は外部公開しない

```go
// Good
func (s *ProductService) GetByID(ctx context.Context, id string) (*Product, error)

// Bad
func (s *ProductService) GetByID(id string) *Product
```

### Vue.js

- Composition API (`<script setup>`) を使用
- コンポーネント名はPascalCase
- props/emitsは明示的に定義

### DynamoDB

- Single Table Design を採用
- PK/SKにはプレフィックスを付ける（例: `USER#`, `PRODUCT#`）
- タイムスタンプはISO 8601形式（RFC3339）

## DynamoDBテーブル設計

テーブル名: `DynamoDBShop`

| エンティティ | PK | SK |
|------------|----|----|
| ユーザー | `USER#<id>` | `PROFILE` |
| 商品 | `PRODUCT#<id>` | `METADATA` |
| カート | `USER#<id>` | `CART#<productId>` |
| 注文 | `USER#<id>` | `ORDER#<orderId>` |
| 注文明細 | `ORDER#<id>` | `ITEM#<productId>` |
| 価格履歴 | `PRODUCT#<id>` | `PRICE#<timestamp>` |
| 在庫ログ | `PRODUCT#<id>` | `INVLOG#<timestamp>` |
| 行動ログ | `USER#<id>` | `ACTIVITY#<timestamp>` |

## 開発コマンド

```bash
# Backend
cd backend
go run cmd/api/main.go

# Frontend
cd frontend
npm run dev

# DynamoDBテーブル作成
./infrastructure/scripts/create-table.sh
```

## 環境変数

### Backend (.env)

```
AWS_REGION=ap-northeast-1
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
DYNAMODB_TABLE=DynamoDBShop
JWT_SECRET=xxx
SERVER_PORT=8080
```

### Frontend (.env)

```
VITE_API_URL=http://localhost:8080/api/v1
```

## API設計

ベースURL: `/api/v1`

- 認証: JWT Bearer Token
- レスポンス形式: JSON
- エラーレスポンス: `{"error": "message"}`

## セキュリティルール

**重要**: 以下の情報は絶対にコミットしないこと

### コミット禁止対象

- APIキー（AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY など）
- パスワード、シークレット（JWT_SECRET, DB_PASSWORD など）
- 秘密鍵ファイル（*.pem, *.key, *.p12）
- 認証情報ファイル（credentials.json, secrets.json, service-account*.json）
- .envファイル（.env.example は除く）

### 安全なコーディング

```go
// BAD - ハードコーディング禁止
const apiKey = "AKIAIOSFODNN7EXAMPLE"

// GOOD - 環境変数から取得
apiKey := os.Getenv("AWS_ACCESS_KEY_ID")
```

```javascript
// BAD
const API_KEY = "sk-xxxxxxxxxxxxx";

// GOOD
const API_KEY = import.meta.env.VITE_API_KEY;
```

### pre-commitフック

機密情報の誤コミットを防ぐため、pre-commitフックを有効化すること：

```bash
git config core.hooksPath .githooks
```

### サンプルファイルの作成

環境変数のサンプルファイル（.env.example）には実際の値ではなくプレースホルダーを使用：

```
AWS_ACCESS_KEY_ID=your-access-key-here
AWS_SECRET_ACCESS_KEY=your-secret-key-here
JWT_SECRET=your-jwt-secret-here
```

## 注意事項

- AWS認証情報は.envに記載し、.gitignoreで除外済み
- DynamoDBはオンデマンドモード（PAY_PER_REQUEST）を使用
- TTLは行動ログ（ACTIVITY）のみに適用（30日で自動削除）

---
> Source: [hosokawa-y/dynamodb-shop](https://github.com/hosokawa-y/dynamodb-shop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
