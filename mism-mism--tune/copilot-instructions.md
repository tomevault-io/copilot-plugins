## knowledge-management

> ドキュメント鮮度・知識管理ルール。リポジトリが唯一の記録システム。


# 知識管理ルール

リポジトリ内でバージョン管理されたアーティファクトだけがエージェントの認識対象になる。
Slack や口頭で決めた設計判断もリポジトリに残す。

## 原則

1. **リポジトリが記録システム** — 設計判断・方針変更は `docs/` にコミットする
2. **AGENTS.md は地図** — 100行以内を維持し、詳細は参照先に委ねる
3. **CLAUDE.md は信頼できるソース** — ドメインモデル・設計方針の単一情報源

## ドキュメント更新が必要な場面

| 変更の種類 | 更新対象 |
|---|---|
| 新しいドメインモデル/エンティティの追加 | `CLAUDE.md` のドメインモデルセクション |
| レイヤー構造・パッケージの変更 | `docs/ARCHITECTURE.md` |
| 新しいルール/スキル/エージェントの追加 | `AGENTS.md` の一覧テーブル |
| UI 文言・ラベルの変更 | `docs/ux-copy-and-labels.md` |
| デザイン仕様の俯瞰・トークン／画面ルールの統合更新 | `docs/design-specification.md` |
| 設計判断・トレードオフの記録 | `docs/design-overview.md` |
| タスクの完了・追加 | `docs/development-tasks.md` |

## 禁止事項

- ドキュメントを更新せずにドメインモデルを変更しない
- `AGENTS.md` に存在しないファイルへの参照を残さない
- `CLAUDE.md` と矛盾する実装を放置しない（コードかドキュメントのどちらかを修正する）

---
> Source: [mism-mism/tune](https://github.com/mism-mism/tune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
