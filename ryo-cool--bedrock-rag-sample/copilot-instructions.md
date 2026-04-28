## bedrock-rag-sample

> description: バックエンド (Golang) のコーディングルール (Backend Coding Rules)

description: バックエンド (Golang) のコーディングルール (Backend Coding Rules)
globs: ["backend/**/*.*"] # Apply to files under the backend directory

---

## 全体方針 (Overall Philosophy)

- **Effective Go:** [Effective Go](mdc:https:/go.dev/doc/effective_go) のガイドラインに従う。
- **Simplicity:** シンプルで読みやすいコードを心がける。
- **Concurrency:** Go の並行処理機能を適切に利用するが、複雑になりすぎないように注意する (`sync` パッケージ、チャネル)。
- **Error Handling:** 明示的なエラーハンドリングを徹底する。
- **Testing:** テスト可能なコードを意識し、ユニットテストを記述する。

## ディレクトリ構造 (Directory Structure)

プロジェクトの規模に応じて、レイヤードアーキテクチャや Clean Architecture を参考に構成する。例:

- `cmd/server/`: main パッケージ、アプリケーションのエントリーポイント
- `internal/`: プロジェクト内部でのみ使用されるコード
  - `handler/` (or `controller/`): HTTP リクエストの処理、レスポンス生成 (Echo handlers)
  - `service/` (or `usecase/`): ビジネスロジック
  - `repository/`: データ永続化層 (DB, S3 など) の抽象化インターフェースと実装
  - `domain/` (or `entity/`): ビジネスドメインのコアとなる構造体やロジック
  - `config/`: 設定の読み込み・管理
- `pkg/`: プロジェクト外部にも公開可能な共有ライブラリ (慎重に使用)
- `api/` (or `schema/`): API スキーマ定義 (OpenAPI など)

## フレームワーク (Framework - Echo)

- **Middleware:** ロギング、リカバリ、認証、CORS など、共通処理はミドルウェアとして実装・適用。
- **Routing:** `echo.Group` を使用してルートをグループ化し、整理する。
- **Request Binding:** `c.Bind()` を使用してリクエストボディやパラメータを構造体にバインドする。
- **Validation:** バリデーションは `validator` などを Echo に統合して使用する。
- **Response:** 適切な HTTP ステータスコードと構造化されたレスポンス (JSON) を返す。

## エラーハンドリング (Error Handling)

- Go 標準の `error` インターフェースを基本とする。
- エラーの原因を追跡するために、`fmt.Errorf` の `%w` verb を使用してエラーをラップする。
- 複数のエラーを扱う場合は `errors.Join` を検討する。
- 特定のエラー種別を判別する必要がある場合は、`errors.Is` や `errors.As` を使用する。
- エンドユーザーに見せるエラーメッセージと、開発者向けのログ出力エラーは区別する。
- HTTP ハンドラー層でエラーを集約し、適切な HTTP ステータスコードに変換する。

## ロギング (Logging)

- 標準ライブラリの `slog` (Go 1.21+) または構造化ロギングライブラリ (例: `zap`, `zerolog`) を推奨。
- ログレベル (Debug, Info, Warn, Error) を適切に使い分ける。
- リクエスト ID などのコンテキスト情報をログに含め、トレースしやすくする。
- エラー発生時には、エラー内容とスタックトレース (必要に応じて) を記録する。
- 個人情報や機密情報はログに出力しない。

## 設定管理 (Configuration Management)

- 環境変数から設定を読み込むことを基本とする。
- 必要に応じて設定ファイル (YAML, TOML) や AWS Parameter Store/Secrets Manager を利用する。
- 設定読み込みライブラリ ([Viper](mdc:https:/github.com/spf13/viper) など) の利用も検討。
- 設定値を格納する構造体を定義する (`config/` ディレクトリなど)。

## 外部サービス連携 (External Services - AWS SDK)

- AWS SDK for Go V2 ([`aws-sdk-go-v2`](mdc:https:/github.com/aws/aws-sdk-go-v2)) を使用する。
- 各 AWS サービス (S3, Bedrock, Textract) とのやり取りを行うクライアントを実装する (`repository/` や専用のパッケージ)。
- クライアントはインターフェースを定義し、依存性注入可能にする。
- SDK のエラーハンドリング、リトライ設定を適切に行う。
- `context.Context` を適切に引き回す。
- SDKの実装例はこちら（https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/gov2/bedrock）迷ったら参照すること。

## リポジトリパターン (Repository Pattern)

- データストア (DB, S3 など) へのアクセスロジックを抽象化するためにリポジトリパターンを適用する。
- `internal/repository/` にインターフェースを定義し、具体的な実装 (例: `s3_repository.go`, `dynamodb_repository.go`) を作成する。
- Service 層は Repository インターフェースに依存する。

## 依存性注入 (Dependency Injection - DI)

- コンストラクタインジェクションを基本とし、コンポーネント間の依存関係を疎にする。
- インターフェースを活用して依存関係を定義する。
- 大規模なアプリケーションでは、DI コンテナ ([Wire](mdc:https:/github.com/google/wire) など) の導入を検討する。

## テスト (Testing)

- Go 標準の `testing` パッケージを使用する。
- テーブル駆動テスト (`[]struct{ ... }`) を活用してテストケースを整理する。
- Repository や外部サービスクライアントのインターフェースに対してモックを作成し、Service 層や Handler 層のユニットテストを行う ([gomock](mdc:https:/github.com/uber-go/mock), [testify/mock](mdc:https:/github.com/stretchr/testify#mock-package) など)。
- Handler 層のテストには `net/http/httptest` を活用する。
- 重要なビジネスロジックは重点的にテストする。
- 必要に応じてデータベース等を含めたインテグレーションテストを実装する。

## ビルドとデプロイ (Build & Deployment)

- `Dockerfile` を作成し、マルチステージビルドを利用して軽量なコンテナイメージを作成する。
- `scratch` イメージや `distroless` イメージの利用を検討する。
- ビルド時のリンカフラグ (`-ldflags`) を使用してバージョン情報などを埋め込む。

### Docker 関連 (Docker Related)

- **マルチステージビルド:** 開発用と本番用を分けたマルチステージビルドを使用する。
- **ベストプラクティス:**
  - `.dockerignore` ファイルを使用して不要なファイルをビルドコンテキストから除外する。
  - 非 root ユーザーでアプリケーションを実行する。
  - 環境変数を適切に利用する。
  - 機密情報はコンテナに埋め込まない。
- **ローカル開発:** 
  - `docker-compose.yml` を使用してローカル開発環境を構築する。
  - 開発時はホットリロード (`air` など) を活用する。
- **イメージタグ:** 
  - Git コミットハッシュや意味のあるバージョンタグを使用する (`v1.0.0`, `v1.0.0-rc1` など)。
  - `latest` タグの使用は本番環境では避ける。
- **セキュリティ:**
  - 定期的に `docker scan` や脆弱性スキャンツールでイメージをチェックする。
  - ベースイメージを最新の安全なバージョンに保つ。
- **AWS ECR へのデプロイ:**
  - CI/CD パイプラインでイメージをビルドし、ECR にプッシュする。
  - ECR リポジトリのライフサイクルポリシーを設定し、古いイメージを自動削除する。

## コードフォーマット (Code Formatting)

- `gofmt` および `goimports` を実行し、コードスタイルを統一する。
- Linter ([golangci-lint](mdc:https:/github.com/golangci/golangci-lint)) を導入し、静的解析を行うことを強く推奨。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ryo-cool) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
