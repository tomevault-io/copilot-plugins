## zenn-template

> このファイルはAIエージェント（Claude Code等）の動作設定や指示を記述するためのファイルです。

# AI エージェント設定

このファイルはAIエージェント（Claude Code等）の動作設定や指示を記述するためのファイルです。

## 設定内容

### 記事作成時の指示（例）

- 技術記事は具体的なコード例を含める
- タイトルは分かりやすく、検索されやすいキーワードを含める
- 記事の構成：問題提起 → 解決方法 → 実装例 → まとめ

### その他

- Zenn記事のFrontmatterは適切に設定
- Publicationにいれるためには`publication_name`を指定
- 画像は相対パスで参照
- 外部リンクは別タブで開くよう設定

## 推奨運用フロー

### オプション1: Publicationレビューフロー
1. `published: false`で記事作成・執筆
2. mainブランチにマージ
3. Zenn上で下書きとして反映
4. Publication機能でチームメンバーにプレビュー共有
5. レビュー後`published: true`に変更して公開

### オプション2: GitHub PRレビューフロー
1. ブランチを切って記事作成・執筆
2. GitHub上でPull Request作成
3. チームメンバーがGitHub上でレビュー
4. `published: true`に設定してマージ・公開

## カスタマイズ

チーム・個人の要件に応じてこのファイルを編集してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tawachan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tawachan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
