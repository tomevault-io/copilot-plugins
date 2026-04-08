## cc-jump-quest

> - このプロジェクトでは、全ての実装ログが `_docs/templates/` に `yyyy-mm-dd_機能名.md` という形式で保存されています。起動時には必ず必ず必ず `_docs/` 配下をコンテキストとして全て読み込んでおき、前回の設計意図や副作用を踏まえた上で提案するようにしてください。

## 実装ログ運用ルール

- このプロジェクトでは、全ての実装ログが `_docs/templates/` に `yyyy-mm-dd_機能名.md` という形式で保存されています。起動時には必ず必ず必ず `_docs/` 配下をコンテキストとして全て読み込んでおき、前回の設計意図や副作用を踏まえた上で提案するようにしてください。
- 実装が完了したら、`_docs/templates/yyyy-mm-dd_機能名.md` というファイル名で実装ログを残すこと。機能名が複数単語の場合はケバブケースを使用するように。（例：2025-01-01_product-feature-implementation.md）
- 実装ログの「日付」欄は **TIME MCP Server** で取得した日時、またはユーザ環境の `now` エイリアス（`date "+%Y-%m-%d %H:%M:%S"`）の出力を使用すること。エイリアスが未設定の場合は `.zshrc` 等に `alias now='date "+%Y-%m-%d %H:%M:%S"'` を追加しておく。
  - 実装ログに含める項目: 実装の目的・背景 / 主な実装内容 / 設計意図 / 副作用 / 関連ファイル

### 実装ログのテンプレート例:

```md
機能名: <ここに機能名>

- 日付: yyyy-mm-dd
- 概要: <実装の目的・背景>
- 実装内容: <主な実装内容>
- 設計意図: <なぜこの設計にしたのか>
- 副作用: <懸念事項があれば明記>
- 関連ファイル: <src/... など>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/camoneart)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/camoneart)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
