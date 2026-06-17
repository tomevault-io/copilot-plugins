## architecture

> DDD・Clean Architecture・依存方向の遵守


# アーキテクチャルール

## 依存の方向

```
UI → Application → Domain ← Infrastructure
```

- ドメイン層は **外部ライブラリの import 禁止**（pure TypeScript のみ）
- インフラ層はドメインのインターフェースを実装し、ドメインに依存する
- Application は Domain と Infrastructure の橋渡し。Domain のリポジトリインターフェースを inject で受け取る

## ドメイン層（domain/）

- `import` はプロジェクト内のみ。`node_modules` からの import は禁止
- エンティティ・値オブジェクト・ドメインサービスは純粋な TypeScript で記述
- リポジトリはインターフェースのみ（`IUserRepository` 等）。実装は infrastructure に配置

## アプリケーション層（application/）

- ユースケースはコンストラクタインジェクションでリポジトリ・サービスを受け取る
- API Route や UI はユースケースを呼ぶだけ。ビジネスロジックを API/UI に書かない

## インフラ層（infrastructure/）

- SQLite、Claude API 等の具体的な技術を隠蔽
- 差し替え可能に保つ（例: SQLite → PostgreSQL）

詳細は [CLAUDE.md](../../CLAUDE.md) の「設計方針」「ディレクトリ構成」を参照。

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
