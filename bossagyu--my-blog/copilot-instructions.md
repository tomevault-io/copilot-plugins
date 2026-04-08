## my-blog

> このファイルは、Claude Code（claude.ai/code）がこのリポジトリで作業する際のガイダンスを提供します。

# CLAUDE.md

このファイルは、Claude Code（claude.ai/code）がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

Hugo静的サイトジェネレーターによるブログ（日本語/英語バイリンガル対応）。テーマ: `hugo-theme-stack`（`themes/`内のgit submodule）。サイト: https://bossagyu.com

## コマンド

```bash
# ローカル開発サーバー
hugo server

# 本番用ビルド
hugo --minify

# 下書きを含めてビルド
hugo server -D

# 新規記事の作成
hugo new blog/NNN-slug-name/index.md
```

npm/yarn/package.jsonはなし — 純粋なHugoプロジェクトです。

## コンテンツ構造

記事は `content/blog/NNN-topic-name/` に連番（001〜047+）で配置されます。各ディレクトリには以下を含みます:
- `index.md` — 日本語記事（メイン）
- `index.en.md` — 英語翻訳（すべての記事に必須）

## 記事のフロントマター

TOML形式で `+++` 区切りを使用:

```toml
+++
title = "日本語タイトル"
description = "SEO用の説明文（120-160文字）"
date = 2026-02-07T00:00:00+09:00
draft = false
categories = ["Engineering"]  # Engineering | Management | Product | Blog
tags = ["tool-name", "topic"]  # 英語で記載、既存のタグと表記を合わせる（下記タグ一覧参照）
+++
```

## SEO description のルール

日本語・英語の両方の記事に `description` フィールドを設定すること。これはGoogle検索結果に表示される説明文として使用される。

**description の書き方:**
- **文字数**: 120〜160文字（日本語）
- **内容**: 記事の要点を簡潔に説明
- **キーワード**: 記事の主要なキーワードを含める（ツール名、技術名など）
- **読者へのメリット**: 「〜の方法を解説」「〜をサンプルコード付きで紹介」など

**良い例:**
```
description = 'AWS Lambda、API Gateway、S3を使ったLINE Botの実装解説。SAMによるサーバーレス構成とPythonコードを詳しく紹介します。'
```

**避けるべき内容:**
- Markdownの記法（`**太字**`、`` `コード` `` など）
- 「この記事では」などの冗長な書き出し
- 記事本文のコピペ（要約すること）

**注意**: descriptionはmeta tagにのみ使用され、記事カードには表示されない。

## 執筆ガイドライン（AGENTS.mdより）

- 日本語で執筆、です・ます調を使用
- 本文は `## 概要` から始め、`##`/`###` の見出しを使用
- コードブロックには言語指定子を付ける
- 日本語記事と対応する英語記事（`index.en.md`）を必ず作成する
- コンテンツの焦点: 実践的なエンジニアリングチュートリアル、ツールセットアップガイド、マネジメント/スクラム関連トピック

## タグの命名ルール

タグを追加する際は、既存の記事のタグと表記を統一すること。新しいタグを追加する前に `grep -h "^tags" content/blog/*/index.md | sort -u` で既存タグを確認する。

**命名規則:**
- 公式ブランド名に従う（例: `VSCode`, `GitHub`, `ChatGPT`, `IntelliJ`）
- プログラミング言語は先頭大文字（例: `Python`, `Go`, `TypeScript`）
- 略語・頭字語は大文字（例: `AWS`, `SEO`, `API`, `S3`）

**主要タグ一覧（正式表記）:**
`AWS`, `Lambda`, `S3`, `API Gateway`, `ChatGPT`, `Codex`, `GitHub Copilot`, `Copilot`, `Git`, `Go`, `gRPC`, `Hugo`, `IntelliJ`, `ITIL`, `LINE BOT`, `Node.js`, `OpenAI`, `Python`, `Redis`, `Scrum`, `SEO`, `TypeScript`, `VSCode`

## 記事の統合・削除時のルール

公開済みの記事を統合（マージ）または削除する場合、旧URLへのアクセスが404にならないよう、統合先の記事のフロントマターに `aliases` を設定すること。

```toml
# 例: 010-favicon を 001-hugo-netlify-build に統合した場合
aliases = ['/blog/010-favicon/']
```

英語版にも対応する `aliases` を設定する:
```toml
aliases = ['/en/blog/010-favicon/']
```

## 技術記事の検証フロー

技術的な記事（ツールのセットアップ手順、コマンドの使い方など）を作成・更新した場合、公開前に以下を確認する:

1. **コマンドの正確性**: 記載したコマンドが実際に動作するか、ターミナルで確認する
2. **公式ドキュメントとの整合性**: 設定値やオプションが公式ドキュメントと一致しているか確認する
3. **バージョン依存性**: 特定バージョンに依存する手順がある場合、対象バージョンを明記する

## 設定

Hugo設定は `config/_default/` に分割:
- `config.toml` — ベースURL、言語設定（日本語がデフォルト、en-US）、テーマ、著者
- `params.toml` — サイドバー、ウィジェット、画像処理、コメント（無効）
- `markup.toml` — コードハイライト、目次設定
- `menu.toml` — ソーシャルリンク

## デプロイ

- **GitHub Pages**: `main`へのプッシュでGitHub Actionsが実行、Hugo 0.120.3 extended、`gh-pages`ブランチにデプロイ
- CDN: Fastly（80+ POP worldwide）
- カスタムドメイン: bossagyu.com（Netlify DNS で A/AAAA レコード管理）
- HTTPS: GitHub Pages 自動管理（Let's Encrypt）

## Gitワークフロー

作業は以下の手順で行うこと:

1. **作業開始前に `main` を最新化する**: `git checkout main && git pull`
2. **フィーチャーブランチを作成する**: `git checkout -b <type>/<description>`
   - ブランチ名の例: `feat/add-new-article`, `fix/redirect-404`, `chore/fix-tags`
3. **変更をコミットする**: コミットメッセージは `<type>: <description>` 形式（例: `feat: 新しい記事を追加`）
4. **リモートにプッシュしてPRを作成する**: `git push -u origin <branch-name>` → `gh pr create`

**注意:**
- `main`に直接コミットしない
- テーマはgit submodule — `git clone --recurse-submodules`または`git submodule update --init`を使用

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bossagyu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bossagyu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
