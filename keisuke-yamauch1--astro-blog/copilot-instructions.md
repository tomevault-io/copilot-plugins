## astro-blog

> このファイルは、このリポジトリのコードを扱う際のClaude Code (claude.ai/code) へのガイダンスを提供します。

# CLAUDE.md

回答は日本語でお願いします。
このファイルは、このリポジトリのコードを扱う際のClaude Code (claude.ai/code) へのガイダンスを提供します。

## プロジェクト概要

これは TypeScript で構築された Astro ベースのブログで、ブログ記事と日記エントリーの両方を特徴とします。コンテンツはこのリポジトリの Content Collections（`src/content/`）で管理され、記事の作成・編集は自作 CMS（blog-cms・https://cms.kechiiiiin.com ・別リポジトリ）が GitHub API 経由で main に直接 commit します。サイトは以下をサポートします：
- **3つのコンテンツタイプ**：blog、diary、emonicle
- **format: html 記事**：microCMS 時代から移行した HTML 本文の記事（`source: 'microcms'` が分岐キー・現役）
- カスタム rehype プラグインを使用した Markdown/MDX コンテンツ作成
- サイトマップとRSSフィードによる SEO 最適化
- TailwindCSS によるダークモードサポート
- ローカル検索機能
- ソーシャルメディア統合と埋め込み（YouTube、Twitter）

## 開発コマンド

```bash
# 開発サーバーを起動
npm run dev

# 本番用ビルド
npm run build

# 本番ビルドのプレビュー
npm run preview
```

## テスト

このプロジェクトは2種類のテストをサポートします：

### ユニットテスト（Vitest）
コンテンツ変換ロジックやユーティリティ関数のテスト。

```bash
# テストを監視モードで実行
npm run test

# テストUIを起動
npm run test:ui

# テストを1回実行
npm run test:run

# カバレッジレポートを生成
npm run test:coverage
```

テストファイルは `src/**/*.test.ts` に配置されます。

### E2Eテスト（Playwright MCP）
ブラウザを使った実際のユーザー操作のテスト。

```bash
# E2Eテストを実行（ヘッドレスモード）
npm run test:e2e

# ヘッド付きモードで実行（ブラウザ表示）
npm run test:e2e:headed

# UIモードで実行（インタラクティブ）
npm run test:e2e:ui

# デバッグモードで実行
npm run test:e2e:debug

# テストレポートを表示
npm run test:e2e:report

# テスト自動生成ツールを起動
npm run test:e2e:codegen

# すべてのテストを実行
npm run test:all
```

テストファイルは `tests/e2e/**/*.spec.ts` に配置されます。

### テスト運用フロー

#### 機能追加後の動作確認
1. 機能を実装
2. ユニットテストを作成・実行（`npm run test:run`）
3. **Playwright MCPを使って動作確認**
   - Claude Codeで「この機能をPlaywrightでテストして」と依頼
   - MCPが自動的にブラウザを起動してテストを実行
   - 動作を目視確認できる
4. 必要に応じてE2Eテストを追加（`tests/e2e/` に配置）

#### E2Eテストの自動生成
Playwright Codegenツールを使用してテストを自動生成できます：

```bash
# 開発サーバーを起動（別ターミナル）
npm run dev

# Codegenツールを起動
npm run test:e2e:codegen
```

ブラウザが開き、操作を記録してテストコードを自動生成します。

#### リグレッションテスト
重要な機能には必ずE2Eテストを追加し、定期的に実行：

```bash
# すべてのテストを実行
npm run test:all
```

### Playwright MCPの使用方法

Playwright MCPはClaude Codeに統合されており、以下のように使用できます：

1. **自然言語でテスト実行を依頼**
   ```
   「管理画面で日記を作成するフローをテストして」
   ```

2. **スクリーンショット取得**
   ```
   「ホームページのスクリーンショットを撮って」
   ```

3. **要素の確認**
   ```
   「ブログ一覧ページに記事が表示されているか確認して」
   ```

4. **インタラクティブなデバッグ**
   ```
   「日記作成フォームの入力をステップバイステップで確認して」
   ```

Playwright MCPは `~/.claude.json` に設定されており、Claude Code起動時に自動的に読み込まれます。

### テストファイル構成

```
tests/
└── e2e/
    ├── home.spec.ts              # ホームページ
    └── blog/
        └── list.spec.ts          # ブログ一覧
```

### テスト設定

- **ヘッド付きモード**: デフォルトで有効（`playwright.config.ts`）
- **スローモーション**: 500ms（視認性向上）
- **自動サーバー起動**: テスト実行時に `npm run dev` を自動実行
- **失敗時の記録**: スクリーンショット、ビデオ、トレースを自動保存

## サイト設定

メインのサイト設定は `src/config.ts` にあります：
- サイトメタデータ（タイトル、説明、URL）
- ソーシャルメディアリンク
- ホームページ記事フィルタリング
- Google Analytics 統合
- 検索機能トグル

## コンテンツ管理（blog-cms）

記事の作成・編集は自作 CMS **blog-cms**（別リポジトリ・Cloudflare Pages）で行う。

- **URL**: https://cms.kechiiiiin.com （Cloudflare Access で保護）
- **仕組み**: CMS が GitHub API でこのリポジトリの `main` に直接 commit → Vercel が自動デプロイ
- **注意**: CMS 起因で remote が先行することがあるため、push 前に必ず `git fetch origin` で確認すること
- **microCMS は卒業済み**（2026-07）: 全記事を `format: html` の md として移行済み。API キーは削除済み。
  旧プレビュー機能（`/preview`・`src/lib/microcms.ts` など）は削除済み

## コンテンツ構造

### ブログ記事（`src/content/blog/`）
- 番号付きフォーマット：`00001_title.md`
- 必須フロントマター：`id`、`title`、`date`
- オプション：`description`、`tags`、`image`、`draft`
- `.md` と `.mdx` 両方のファイルをサポート

### 日記エントリー（`src/content/diary/`）
- 日付ベースフォーマット：`YYYY-MM-DD_title.md`
- 必須フロントマター：`title`、`date`
- オプション：`weather`、`image`

## 主要アーキテクチャ

### コンテンツコレクション（`src/content/config.ts`）
Zod バリデーションを使用したブログ、日記、プロフィールコレクションのスキーマを定義。

### 記事フィルタリング（`src/utils/posts.ts`）
以下を含む中央集約的な記事フィルタリングロジック：
- 公開記事フィルタリング（下書きと未来の日付を除外）
- ホームページ用のタグベースフィルタリング
- 日付ソート
- 記事制限機能

### レイアウト
- `BaseLayout.astro`：SEO メタタグを持つメインページ構造
- `BlogPost.astro`：ブログ記事レイアウト
- `DiaryPost.astro`：日記エントリーレイアウト

### カスタム Rehype プラグイン（`astro.config.mjs` 内）
- `rehypeTargetBlank`：すべてのリンクを新しいタブで開く
- `rehypeImageNotProse`：Tailwind Typography 用に画像に `not-prose` クラスを追加
- `rehypeCodeTitle`：`language:title` 構文によるコードブロックタイトルをサポート
- `rehypePrettyCode`：GitHub Dark テーマでコードシンタックスハイライト

### カスタム Remark プラグイン（`astro.config.mjs` 内）
- `remarkBreaksForDiary`：日記コレクションのみに `remark-breaks` を適用し、単一の改行（`\n`）を `<br>` タグとして表示

### マークダウン改行処理の特殊設定

日記とブログで異なる改行処理を行います：

**日記エントリー（`src/content/diary/`）**：
- 単一の改行（`\n`）が `<br>` タグとして表示される
- `remark-breaks` プラグインを適用

**ブログ記事（`src/content/blog/`）**：
- CommonMark標準に従う
- 単一の改行は無視される（スペースとして扱われる）
- 改行するには、行末にスペース2個+改行、またはダブル改行（段落区切り）が必要

**実装詳細**：
- `astro.config.mjs` の `remarkBreaksForDiary` カスタムプラグイン
- ファイルパスに `/content/diary/` が含まれる場合のみ `remark-breaks` を適用

### コンテンツ処理パイプライン
1. Markdown/MDX ファイルは Astro のコンテンツコレクションを通じて処理されます
2. `src/content/config.ts` で Zod スキーマがフロントマターを検証します
3. `src/utils/posts.ts` 関数を通じてコンテンツがフィルタリングされます：
   - `filterPublishedPosts()`：下書きと未来の日付の記事を除外
   - `filterPosts()`：タグフィルタリングと記事制限
   - `sortPostsByDate()`：時系列ソート
4. Rehype プラグインが HTML 出力を変換します

## ルーティング構造

- `/`：フィルタリングされた記事を持つホームページ
- `/blog/[...page]`：ページ分割されたブログ一覧
- `/blog/[slug]`：個別ブログ記事
- `/diary/[...page]`：ページ分割された日記一覧
- `/diary/[year]/[month]/[day]`：個別日記エントリー
- `/tags/[tag]`：タグベース記事フィルタリング
- `/archive`：アーカイブページ
- `/profile`：プロフィールページ

## 開発ノート

- サイト言語は日本語に設定（`lang="ja"`）
- Typography プラグインを含む TailwindCSS を使用
- 統合：Tailwind、MDX、Sitemap、Astro Embeds
- GitHub Dark テーマによる `rehype-pretty-code` でのコードシンタックスハイライト
- 従来の MPA 動作向けに設定（View Transitions 無効）

## データフローアーキテクチャ

### 記事公開ワークフロー
1. blog-cms（cms.kechiiiiin.com）から記事を作成（または `src/content/` に直接 md を追加）
2. `draft: false` に設定し、`date` が未来でないことを確認
3. 記事は `isPublished()` と `draft` ステータスによって自動的にフィルタリングされます
4. ホームページは `siteConfig.homepage` 設定に基づいてフィルタリングされた記事を表示

### コレクションタイプ
- **ブログコレクション**：タグと説明を持つ番号付き記事（`00001_title.md`）
- **日記コレクション**：天気情報を持つ日付ベースエントリー（`YYYY-MM-DD_title.md`）
- **プロフィールコレクション**：静的コンテンツページ

### 設定依存関係
- `src/config.ts`：サイトメタデータ、ソーシャルリンク、ホームページフィルタリング
- `astro.config.mjs`：ビルド設定、rehype プラグイン、統合
- `src/content/config.ts`：コンテンツスキーマ検証
- `src/utils/posts.ts`：記事フィルタリングとソートロジック

---
> Source: [keisuke-yamauch1/astro-blog](https://github.com/keisuke-yamauch1/astro-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
