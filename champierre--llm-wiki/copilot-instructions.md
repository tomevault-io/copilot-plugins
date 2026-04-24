## llm-wiki

> このリポジトリは石原淳也に関するLLM Wikiです。LLMが継続的に構築・維持するマークダウンベースのナレッジベースです。

# LLM Wiki — 石原淳也

このリポジトリは石原淳也に関するLLM Wikiです。LLMが継続的に構築・維持するマークダウンベースのナレッジベースです。

## ディレクトリ構造

```
content/      # Quartz用コンテンツ（GitHub Pages公開用、wiki/と同内容）
  index.md    # トップページ兼カタログ
  overview.md # 石原淳也の概要
  log.md      # 時系列の操作ログ
  entities/   # 人物・組織・プロジェクトのページ
  concepts/   # 概念・テーマのページ
  sources/    # ソース要約ページ
wiki/         # LLMが生成・管理するWikiページ（Obsidian用）
raw/          # 生のソースドキュメント（不変）
quartz/       # Quartz静的サイトジェネレーター
llm-wiki.md   # LLM Wikiパターンの説明
CLAUDE.md     # このファイル（スキーマ・規約）
```

**重要**: `wiki/` と `content/` は同じ内容を保持する。ページを追加・更新する際は**両方**を更新すること。`content/` はQuartzによるGitHub Pages公開用、`wiki/` はObsidian閲覧用。

## Wiki規約

### ページ形式
- すべてのWikiページはMarkdown形式
- 各ページの先頭にYAMLフロントマターを含める：
  ```yaml
  ---
  title: ページタイトル
  type: entity | concept | source | overview | comparison | synthesis
  tags: [タグ1, タグ2]
  sources: [参照元URL or ファイル名]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  ---
  ```
- ページ間リンクは `[[パス|表示テキスト]]` 形式（Obsidian/Quartz互換）
  - **順序は `[[パス|表示名]]`**（パスが先、表示名が後）
  - パスはcontent/からの相対パス、拡張子なし
  - 例: `[[entities/ml2scratch|ML2Scratch]]`、`[[concepts/programming-education|プログラミング教育]]`
  - 表示名不要の場合は `[[entities/coderdojo]]` のようにパスのみでもよい
  - 対応するページが存在しないリンク（`[[Scratch]]`等）はそのまま残してよい
- 画像は `raw/assets/` に配置

### GitHub Pages
- **公開URL**: https://champierre.github.io/llm-wiki/
- **ツール**: Quartz v4（静的サイトジェネレーター）
- **デプロイ**: GitHub Actions（main pushで自動ビルド・デプロイ）
- **設定**: `quartz.config.ts`（タイトル、ロケール、baseUrl等）

### ページタイプ
- **entity**: 人物、組織、プロジェクトなど固有名詞のページ
- **concept**: 技術、概念、テーマのページ
- **source**: ソースドキュメントの要約ページ
- **overview**: トピックの俯瞰ページ
- **comparison**: 比較・分析ページ
- **synthesis**: 複数ソースを統合した考察ページ

## ワークフロー

### インジェスト
1. `raw/` にソースを配置
2. ソースを読み、主要な情報を抽出
3. `wiki/sources/` にソース要約ページを作成
4. 関連するエンティティ・コンセプトページを作成または更新
5. `wiki/index.md` を更新
6. `wiki/log.md` にエントリを追加

### クエリ
1. `wiki/index.md` を読んで関連ページを特定
2. 該当ページを読んで回答を合成
3. 有用な回答は新しいWikiページとして保存可能

### リント
- 矛盾の検出
- 古い情報の特定
- 孤立ページの発見
- 欠落しているクロスリファレンスの追加
- 新しい調査すべき質問の提案

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/champierre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
