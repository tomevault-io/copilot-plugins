## todo-go

> Claude Code がこのリポジトリで作業する際のガイドです。

# CLAUDE.md

Claude Code がこのリポジトリで作業する際のガイドです。

## プロジェクト概要

Go 製の Todo 管理 REST API。Clean Architecture で構成されたバックエンドサービス。

- **言語**: Go 1.25
- **フレームワーク**: Gin (HTTP), Bob (ORM), Wire + Kessoku (DI)
- **DB**: Neon (PostgreSQL) / Atlas (マイグレーション)
- **ホスティング**: Google Cloud Run (asia-northeast1)
- **CI/CD**: GitHub Actions + Workload Identity Federation
- **ツール管理**: Aqua
- **観測性**: OpenTelemetry, zerolog

## ディレクトリ構成

```
backend/
├── cmd/server/          # エントリポイント・DI設定
├── internal/
│   ├── domain/          # エンティティ・リポジトリインターフェース
│   │   ├── todo/
│   │   ├── user/
│   │   └── apikey/
│   ├── usecase/         # ユースケース層
│   ├── infrastructure/  # DB・キャッシュなど外部依存の実装
│   │   └── rdb/         # Bob ORM モデル・Atlas マイグレーション
│   └── presentation/    # HTTP ハンドラ
└── docs/swagger/        # Swagger 生成ドキュメント

infra/                   # Terraform（GCP + Neon）
.github/
├── actions/
│   ├── setup-gcp/       # WIF認証・gcloud・Docker設定
│   └── setup-go/        # Go環境セットアップ
└── workflows/
    ├── backend-cd.yml   # main push → Cloud Run デプロイ
    └── backend-ci.yml   # PR → lint + test
```

## よく使うコマンド

```bash
cd backend

make build          # バイナリビルド
make run            # サーバ起動
make test           # テスト実行
make lint           # golangci-lint
make air            # ホットリロード起動（開発時推奨）
make up             # Docker Compose でDB起動
make down           # DB停止
```

### コード生成

```bash
make gen-all        # 全ジェネレータ実行（Wire + Kessoku + Bob）
make gen-wire       # Wire DI コード生成
make gen-kessoku    # Kessoku DI コード生成（go generate）
make gen-bob        # Bob ORM モデル生成（bobgen-psql）
make gen-swagger    # Swagger ドキュメント生成
```

### DB マイグレーション

```bash
make migrate-diff name=<name>   # マイグレーションファイル作成
make migrate-apply              # マイグレーション適用
make migrate-hash               # ハッシュ更新
make schema-clean               # スキーマ全削除（破壊的操作）
```

## 開発ワークフロー

1. `make up` でDB起動
2. `make migrate-apply` でマイグレーション適用
3. `make air` でホットリロード起動
4. スキーマ変更時: `migrate-diff` → `migrate-apply` → `gen-bob` → `gen-all`
5. DI 変更時: `gen-all` でコード再生成

## アーキテクチャのルール

- **依存方向**: `presentation → usecase → domain ← infrastructure`
- `domain` 層は外部パッケージに依存しない
- DB アクセスは Bob ORM を通じて `infrastructure/rdb` で行う
- DI は Wire/Kessoku で管理し、手動で依存を渡さない
- API キーは `X-API-Key` ヘッダで認証

## テスト

```bash
make test           # 全テスト実行（約5〜6秒）
```

### テスト方針

層ごとにテスト戦略を分けている。

| 層 | 方式 | DB起動 |
|---|---|---|
| ドメイン（値オブジェクト・エンティティ） | ユニットテスト | 不要 |
| ユースケース | mockery 生成モックを使用 | 不要 |
| リポジトリ | testcontainers で自動起動（`TestMain` で1回） | 自動（Docker必須） |

- ドメイン・ユースケーステストは `make up` 不要でどこでも実行可能
- リポジトリテストは Docker が起動していれば自動でコンテナを起動・終了する
- `make up` はアプリ開発時のみ必要（テストでは不要）

### モック生成

```bash
mockery             # internal/domain 配下のインターフェースからモック生成
```

- 設定: `.mockery.yml`（`package mocks`、非 `_test.go` ファイルで他パッケージからインポート可能）
- 生成先: `internal/domain/**/mocks/mock_*.go`
- インターフェース変更時は `mockery` を再実行する

### テストヘルパー

- `internal/testhelper/db.go`: testcontainers を使った DB 起動・マイグレーション適用・TRUNCATE ヘルパー
- `internal/config/clock/mock_clock.go`: 固定時刻を返す `MockClock`

## インフラ・デプロイ

### 構成概要

```
GitHub Actions (backend-cd.yml)
  1. Docker build → Artifact Registry (asia-northeast1)
  2. Atlas migrate apply → Neon (aws-ap-southeast-1 / シンガポール)
  3. gcloud run deploy → Cloud Run (asia-northeast1)
```

認証は Workload Identity Federation（サービスアカウントキーなし）。

### 初回セットアップ（Terraform）

```bash
cd infra
cp terraform.tfvars.example terraform.tfvars
# terraform.tfvars に実値を記入（gcp_project_id, neon_api_key, neon_org_id）

terraform init
terraform apply
```

apply 後に GitHub Variables / Secrets を設定する:

```bash
terraform output github_variables   # → GitHub Settings > Variables に設定
terraform output -json github_secrets  # → DATABASE_URL を Secrets に設定
```

### 環境変数

| 変数 | ローカル | 本番 (Cloud Run) |
|---|---|---|
| `DB_HOST` | localhost | Neon エンドポイント（Terraform 管理） |
| `DB_SSL_MODE` | 未設定（disable） | `require` |
| `PORT` | 8080（デフォルト） | Cloud Run が自動セット（設定不可） |
| `APP_ENV` | local | production |

`PORT` は Cloud Run の予約済み変数。Terraform・CDワークフロー共に設定してはいけない。

### CD トリガー

`backend/` 配下の変更を `main` にプッシュすると `backend-cd.yml` が自動起動する。
マイグレーションファイルの変更も同じフローで本番 Neon に適用される。

### Terraform の注意点

- `infra/terraform.tfvars` は gitignore 済み。絶対にコミットしない
- Neon 無料プランの制約: `history_retention_seconds` 上限 21600秒（6時間）
- Neon 利用可能リージョン: `aws-ap-southeast-1`（東京 `aws-ap-northeast-1` は不可）
- Cloud Run のイメージは `lifecycle.ignore_changes` で CD に委ねている

## フロントエンド

Next.js 16（App Router）+ TailwindCSS v4 + shadcn/ui（@base-ui/react ベース）。

### コマンド

```bash
cd frontend

npm run dev    # 開発サーバー起動 (localhost:3000)
npm run build  # プロダクションビルド
```

バックエンドが必要な場合は別ターミナルで `cd backend && make air` を先に起動する。

### 環境変数（`frontend/.env.local`）

| 変数 | 説明 |
|---|---|
| `BACKEND_URL` | Go バックエンドのURL（デフォルト: `http://localhost:8080`） |
| `BACKEND_API_KEY` | Go API の `X-API-Key`（サーバー側のみ使用。ブラウザ非公開） |

`BACKEND_API_KEY` の取得方法：バックエンドを起動後、`POST /v1/users` → `POST /v1/api-keys` で発行する。

### ディレクトリ構成

```
frontend/
├── app/
│   ├── api/todos/         # Next.js Route Handlers（Go API プロキシ）
│   ├── todos/             # Todo リスト・詳細
│   ├── settings/          # 設定画面
│   ├── providers.tsx      # TanStack Query プロバイダー
│   └── layout.tsx         # ルートレイアウト
├── components/
│   ├── layout/            # Sidebar, MobileNav, AppShell
│   └── todo/              # TodoCard, TodoList, TodoCreate, TodoPanel, EmptyState
├── lib/api.ts             # クライアント側 fetch ラッパー（/api/* を呼ぶ）
├── types/todo.ts          # TypeScript 型定義
└── DESIGN.md              # Google DESIGN.md 形式のデザイン仕様
```

### 注意点

- shadcn/ui は `@base-ui/react` ベースで、従来の Radix UI と異なる。`asChild` prop が使えない
- `NEXT_PUBLIC_` なし変数はブラウザに公開されない（API キーは必ず非公開で使う）
- `PORT` 環境変数は Cloud Run が予約済み（設定不可）

### Firebase Auth（未実装・別計画あり）

Firebase の設定が可能になった時点で実施する。
計画: `~/.claude/plans/google-cloud-api-cloudrun-githubactions-serene-adleman.md` の「別計画」セクション参照。

## Claude Code ワークフロー指針

- 複雑なタスクは `/plan` で設計してから実装する
- コンテキストが 30〜40% に達したら `/compact` を実行する
- 複数ステップの実装はタスクリストで進捗管理する
- 問題の診断には `/doctor` を使う
- コミットはファイル単位で分けて作成する（複数ファイルをまとめない）

### Agent ツール（サブエージェント）の使用ルール

- **Agent ツールはユーザーが「並列で探索して」「サブエージェントを使って」など明示的に指示した場合のみ使用する**
- 自律的な並列探索・調査目的での spawn は禁止
- Agent を使う前に、使う理由と何を委託するかをユーザーに説明して承認を得ること

### セッション分割の目安

以下のような大きなタスクは、必ずセッションを分けて着手する:

- インフラ（Terraform / GCP）の変更
- フロントエンドの大規模変更
- バックエンドの大規模変更（新エンティティ追加など）
- 複数レイヤー（DB + usecase + presentation）にまたがる機能追加

### Issue 対応フロー

**小（1ファイル・1層・数分）**
```
実装 → コミット → /clear
```

**中（1ドメイン・複数ファイル）**
```
/clear → 「Issue #X を対応して」→ 実装・コミット
→ 「やったこと・次のアクションを /memory に保存して」→ /clear
```

**大（複数ドメイン跨ぎ）**
```
セッション1: /plan で全体設計 → /clear
セッション2: バックエンド実装 → コミット → /clear
セッション3: フロントエンド実装 → コミット → /clear
セッション4: インフラ変更 → コミット → /clear
```
- ドメインをまたぐ大タスクは必ず /plan から始める
- 「計画は ~/.claude/plans/[ファイル名]」と次セッションに伝えれば計画を引き継げる
- 進捗はファイルに書かず /memory に保存する（git を汚さないため）
- コンテキストが 30〜40% に達したら /compact

---
> Source: [jimi024rion/todo-go](https://github.com/jimi024rion/todo-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
