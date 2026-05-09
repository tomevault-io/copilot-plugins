## go-claude-code-references

> Cloud Functions (Go 2nd Gen) によるバックエンド。Firestore, Pub/Sub, Genkit を活用。

# Backend

Cloud Functions (Go 2nd Gen) によるバックエンド。Firestore, Pub/Sub, Genkit を活用。

## Setup (初回のみ)

```bash
# Firebase プラグイン（Agent Skills）のインストール
claude plugin marketplace add firebase/agent-skills
claude plugin install firebase@firebase

# MCP サーバーは .mcp.json で自動設定される（Node.js / npx が必要）
```

## Essential Commands

以下のツールが必要。未インストールの場合は `go install` で導入すること。
- `golangci-lint` — リント
- `goimports` — import 整理（PostToolUse hook で自動実行。未導入時は gofmt にフォールバックするが、lint が失敗する）
- `mockery` — モック生成
- `oapi-codegen` — OpenAPI コード生成（`api/openapi.yaml` 作成後に必要）

```bash
task test         # テスト実行
task lint         # golangci-lint 実行
task gen          # コード自動生成 (mockery, oapi-codegen ※api/openapi.yaml 作成後)
task build        # ビルド確認 (go build -o /dev/null ./cmd/...)
task dev          # Firebase Emulators + Air 起動（未実装）
```

## Architecture

Onion Architecture (Light)。詳細は `docs/backend_architecture.md` を参照。

### レイヤー構成

```
cmd/main.go                # エントリーポイント, Manual DI
internal/
  domain/                  # 外部依存なし
    model/                 # Entity（純粋な Go Struct、タグなし）
    repository/            # Repository Interface（メソッド定義のみ）
    errors.go              # Domain Error 定義
  usecase/                 # ビジネスロジックのフロー制御
  handler/                 # 入口/出口の制御
    http/                  # Echo による REST API
      middleware/          # Auth, Logger 等
    trigger/               # Firestore Event → Pub/Sub 発行
    worker/                # Pub/Sub Subscriber
  infra/                   # 外部サービスとの接続
    firestore/             # Firestore Repository 実装 + DTO
    pubsub/                # Pub/Sub Publisher
    genkit/                # LLM (Gemini) 翻訳
    crawler/               # OGP HTML パーサー
```

### レイヤー依存ルール

- domain → 外部依存なし
- usecase → domain の repository interface にのみ依存
- handler → usecase を呼び出す。ステータスコード/リトライ制御はここで判断
- infra → domain の interface を実装。DTO 変換はここで行う

## Code Style

- Manual Mapping のみ（jinzhu/copier 等は禁止）
- DTO に `firestore` タグ付与。Domain Entity にはタグ不要
- `toEntity()` / `toDTO()` を手動実装
- DI は `cmd/main.go` で手動ワイヤリング
- エラーは `domain/errors.go` で定義

## Go Idioms

- エラーラップ: `fmt.Errorf("context: %w", err)` で下位エラーをラップする
- エラー判定: `errors.Is()` / `errors.As()` を使用する（`==` 比較は禁止）
- `context.Context` は全関数の第1引数。struct に格納しない
- Accept interfaces, return concrete types（依存逆転の原則）
- Channel のクローズは sender 側で行う。goroutine のライフタイムは context で制御する
- 名前付き戻り値は defer でのエラーハンドリング時のみ使用する
- Go 1.24 のモダン機能を積極使用する（詳細は `.claude/rules/go-modern.md` 参照）

## Error Handling

### Handler (Worker)
- 致命的エラー（バリデーション違反、`ErrInvalidInput`）→ `return nil`（DLQ へ）
- 一時的エラー（DB接続失敗等）→ `return err`（リトライ）
- 判別不能なエラー → `return err`（リトライ上限後 DLQ）

### UseCase
- 冪等性を保証する。戦略はユースケースごとに異なる（詳細は `.claude/rules/usecase.md` 参照）

## Testing

- Domain/UseCase: mockery でモック、ユニットテスト
- Infra: Firestore Emulator で結合テスト
- テスト命名: `Test_関数名_条件_期待結果`

## Key Design Documents

- `docs/backend_architecture.md` - 基本方針・技術スタック
- `docs/foundation.md` - 基盤実装仕様（Phase 4）
- `docs/usecase_translation.md` - 翻訳処理設計
- `docs/usecase_ogp.md` - OGP 取得設計
- `docs/usecase_db-trigger.md` - db-trigger 設計
- `docs/usecase_languages.md` - 言語管理設計
- `docs/infrastructure_design.md` - インフラ設計
- `docs/firebase_erd.md` - Firestore スキーマ
- `docs/plan_*.md` - 機能実装計画書（`/plan-feature` で生成）

## Commit Message Format

Conventional Commits 形式を使用する。

```
type(scope): subject
```

- **type**: feat, fix, refactor, test, docs, chore
- **scope**: translation, ogp, domain, infra, handler, config, deps
- **subject**: 英語、命令形、小文字開始、末尾ピリオドなし、72文字以内

コミットは `/commit` スキル、PR 作成は `/pr` スキルを使用する。

## Phase 実行ガイド

開発タスクの全体像は `docs/backend_tasks.md` を参照。各タスクに実行者・依存・参照が記載されている。

### Phase 4: 基盤実装

```
/implement-foundation
```

単一セッションで実行する。`docs/foundation.md` が仕様書。

### Phase 5: ユースケース実装

```
/implement-usecase db-trigger
/implement-usecase translation
/implement-usecase ogp
/implement-usecase languages
```

独立したユースケースは worktree で並列実行可能。詳細は `docs/backend_tasks.md` の並列実行の組み合わせを参照。

### 大規模機能の計画

```
/plan-feature <feature-name>
```

4+ PR にまたがる大規模機能の場合に使用する。`docs/plan_<feature-name>.md` に構造化された実装計画書を生成する。
計画書はコールドスタート対応で、別セッションがそのまま実装を開始できる。

- 中小規模（1-3 PR）: `/feature-dev` を直接使用
- 大規模（4+ PR）: `/plan-feature` で計画 → ステップごとに実装

### Phase 6: テスト・デプロイ

テストは Claude Code で実行可能。デプロイ設定は人間が担当。

## Pre-Commit Checklist

1. `task check`（fmt, vet, lint, test, build を一括実行）

## Context Management

### 自動通知

50 tool call 到達時、以降 25 call ごとに `/compact` 推奨メッセージが表示される。
自動コンパクションは context の 60% 到達時にトリガーされる。

### コンパクション判断ガイド

| 状況 | 推奨 | 理由 |
|------|------|------|
| Skill 実行中（Phase 途中） | しない | Phase コンテキストが失われる |
| Skill の Phase 完了直後 | する | 次の Phase は docs から再読み込み可能 |
| `task check` 失敗の修正ループ中 | しない | エラー文脈が必要 |
| `task check` 全パス後 | する | 安全な区切り |
| デバッグ中 | しない | エラートレースが必要 |
| コミット・PR 作成前 | する | 最後の作業に集中するため |

### コールドスタート時の復帰

コンパクション後や新セッション開始時:
1. CLAUDE.md と `.claude/rules/` は自動読み込みされる
2. 実行中の Skill があれば再度呼び出す（例: `/implement-usecase translation`）
3. `git status` と `git diff` で進捗を確認する

## Parallel Execution

複数の Claude Code セッションで並列実装する場合:
- 各セッションは worktree を使用して作業する
- 各セッションは独立したブランチで作業し、個別に PR を作成する
- `cmd/main.go` の DI 部分は並列編集でコンフリクトしやすいため、
  PR マージ時にリベースで解決する

---
> Source: [arai-kazuhiro/go-claude-code-references](https://github.com/arai-kazuhiro/go-claude-code-references) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
