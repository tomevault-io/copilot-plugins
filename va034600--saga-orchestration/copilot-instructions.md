## saga-orchestration

> - このサービスはマイクロサービスで設計する

- このサービスはマイクロサービスで設計する
- テスト名は日本語で記述する

## ディレクトリ構成

```
saga-orchestration/
├── services/          # 言語非依存リソース（openapi.yml, Flyway マイグレーション SQL）
├── kotlin/            # Kotlin + Spring Boot 実装
├── go/                # Go 実装
└── docker/            # Docker Compose, LocalStack, PostgreSQL 初期化
```

- `services/<service>/openapi.yml` — API 定義（言語問わず参照）
- `services/<service>/db/migration/` — Flyway マイグレーション SQL

## サービス構成

| サービス | ポート | 役割 |
|---------|--------|------|
| orchestrator | 8080 | Saga オーケストレーション（同期 / Step Functions 非同期） |
| order-service | 8081 | 注文管理 |
| payment-service | 8083 | 決済管理 |
| compensation-service | 8084 | 補償処理（SQS 経由で非同期実行） |

## 主要パターン

- **Transactional Outbox**: ドメイン操作と外部イベント発行を同一トランザクションで保証
  - orchestrator: `outbox_events` → EventBridge
  - compensation-service: `outbox_tasks` → HTTP 補償呼び出し
- **Saga（同期）**: orchestrator が order → payment → complete を順次呼び出し、失敗時に補償イベントを発行
- **Saga（非同期）**: AWS Step Functions のステートマシンで同等フローを実行

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/va034600)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/va034600)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
