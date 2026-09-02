## growi-news-feed

> このファイルは、`feed.json` にニュースを追加する AI エージェント（PrimaVista の `gnf-add-news` スキル）と人間の共同編集者が従う **入稿規約の原典** である。データフローや全体像は [README.md](README.md) を参照し、ここでは重複させない。本ファイルは「エントリを 1 件足すときに守るべきルール」に絞る。

# CLAUDE.md — growi-news-feed 入稿規約

このファイルは、`feed.json` にニュースを追加する AI エージェント（PrimaVista の `gnf-add-news` スキル）と人間の共同編集者が従う **入稿規約の原典** である。データフローや全体像は [README.md](README.md) を参照し、ここでは重複させない。本ファイルは「エントリを 1 件足すときに守るべきルール」に絞る。

## 編集対象とゴール

- 編集するのは **`feed.json` のみ**（`{ "version": "1.0", "items": [...] }`）。`items` 配列の末尾に新規エントリを 1 件追加する。
- 既存エントリの値は**書き換えない**（特に `id`。理由は後述）。追加のみを原則とする。
- push 後は CI（`validate.yml`）が ajv スキーマ検証＋`id` 一意性チェックを行い、main マージで GitHub Pages へ反映される。

## エントリのスキーマ（`schema/feed.schema.json` が正典）

全階層 `additionalProperties: false`。**未知のフィールドを足すと検証で落ちる**。

| フィールド | 必須 | 制約 |
|---|---|---|
| `id` | ✅ | 一意の文字列。**マージ後は絶対に変更しない**（下記） |
| `title` | ✅ | 多言語オブジェクト。**`ja_JP` 必須**。キーは `^[a-z]{2}_[A-Z]{2}$`、値は 1 文字以上 |
| `publishedAt` | ✅ | ISO 8601（例: `2026-06-17T00:00:00Z`） |
| `emoji` | | 表示アイコン 1 つ（未設定時は 📢 がフォールバック） |
| `body` | | `title` と同じキー規則。値は空文字も可 |
| `url` | | 詳細ページ URL（`format: uri`） |
| `conditions.targetRoles` | | `["admin"]` / `["general"]`。未設定で全ユーザー |
| `conditions.growiVersionRegExps` | | 例 `["^7\\.5\\..*"]`。未設定で全バージョン |

ルートの `version` は `"1.0"` 固定。新規エントリ追加時に変えない。

## `id` の規約（最重要）

- GROWI 本体は `id` を `externalId` として既読管理・重複排除に使う。**マージ後に `id` を変えると、そのニュースを既読にしていた全ユーザーの既読状態がリセット**され「未読」で再表示される。
- そのため `id` は **一度決めたら不変**。typo に後から気付いても修正せず、「新しい `id` で作り直し＋旧エントリ削除」で対処する。
- 命名: 内容由来の安定した **kebab-case スラッグ**（例: `release-v7-5-0`、`maintenance-2026-07`）。日付やバージョンなど後から見て識別しやすい語を含める。
- 追加前に **既存 `feed.json` の全 `id` を読み、衝突しないこと**を必ず確認する。

## 多言語（title / body）

- 推奨ロケールは **`ja_JP` / `en_US` / `zh_CN` / `fr_FR` / `ko_KR`** の 5 つ。`title`・`body` とも 5 言語すべてを記入するのを既定とする。
- `ja_JP` の原文を基準に、他言語は自然な翻訳を入れる（機械的な逐語訳ではなく、各言語として読める文に）。
- `title` の値は空にしない（スキーマ上 1 文字以上必須）。`body` は空文字を許容するが、原則 5 言語そろえる。

## 入稿前のローカル検証（必須）

push / PR の前に、必ずローカルで検証して壊れたフィードを出さない。

```bash
npm ci          # 初回のみ（ajv-cli 等を取得。node_modules は gitignore 済み）
npm run validate
```

`npm run validate` は ajv によるスキーマ検証と `scripts/check-uniqueness.ts` の `id` 一意性チェックを通す。**ここが通らないものは push しない**。

## push 戦略と人間承認

- 反映は **PR 経由**を既定とする（ブランチを切って `feed.json` を変更 → PR）。CI の検証とレビューを経て main にマージ、`pages.yml` がデプロイする。
- ホスティング操作（push / PR）は **`gh` CLI** で行う。
- **人間の明示承認なしに push / PR を作らない**。AI スキルは、生成した全文（5 言語）とメタデータをプレビュー提示し、運用者の承認を得てからコミットする。

## やってはいけないこと

- 既存エントリの `id` の変更、`feed.json` 以外への分割（フィードは単一ファイルに集約する）。
- スキーマ外フィールドの追加（`additionalProperties: false`）。
- 検証を通さずに push、人間承認を飛ばした自動 push。
- クレデンシャルのコミット（Pages 公開後は **Git 履歴も全世界に公開**される）。

---
> Source: [growilabs/growi-news-feed](https://github.com/growilabs/growi-news-feed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
