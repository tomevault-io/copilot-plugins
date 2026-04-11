## zenn-docs

> このドキュメントは、Gemini CLI がこのリポジトリで作業する際のコンテキストと作業方針を定義します。

# Gemini CLI 作業方針

## 目的

このドキュメントは、Gemini CLI がこのリポジトリで作業する際のコンテキストと作業方針を定義します。

## 出力スタイル

### 言語

- **会話言語**: 日本語
- **コード内コメント**: 日本語
- **エラーメッセージ**: 英語

### トーン

- 簡潔で明確な表現を使用
- 技術的に正確な情報を提供
- 不確実な点は明示する

### 形式

- Markdown 形式で整形された出力
- コードブロックには言語指定を含める
- リストや表を活用して情報を整理

## 共通ルール

### 言語使用ルール

- **会話は日本語**で行う
- コード内のコメントは**日本語**で記載
- エラーメッセージは**英語**で記載
- **日本語と英数字の間には半角スペース**を挿入

### コミット・PR ルール

- **コミットメッセージ**: [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従う
  - 形式: `<type>(<scope>): <description>`
  - `<description>` は英語で記載
  - 例: `docs(articles): add article about Docker setup`
- **ブランチ命名**: [Conventional Branch](https://conventional-branch.github.io) に従う
  - 形式: `<type>/<description>`
  - 例: `docs/add-docker-article`

## プロジェクト概要

このリポジトリは、[Zenn.dev](https://zenn.dev/book000) で公開する技術記事・書籍を管理するためのリポジトリです。

### 目的

- 日本語の技術記事・書籍の執筆・管理
- Zenn.dev プラットフォームでの公開

### 主な機能

- Markdown 形式での記事作成
- textlint による日本語文章の品質管理（170+ ルール）
- markdownlint による Markdown 構文の品質管理（120+ ルール）
- プレビューサーバーでの確認（zenn-cli）

### ディレクトリ構造

```text
.
├── articles/           # 技術記事（Markdown ファイル）
├── books/             # 書籍コンテンツ
├── .textlint/         # textlint カスタムルール
├── .textlintrc        # textlint 設定
└── .markdownlint.jsonc # markdownlint 設定
```

## コーディング規約

### フォーマット

- **Markdown**: markdownlint ルールに従う
  - 見出しは ATX スタイル (`# Header`)
  - リストはダッシュスタイル (`-`) で 2 スペースインデント
  - コードブロックは言語指定必須

### 命名規則

- **ファイル名**: `kebab-case.md`（例: `docker-setup-guide.md`）
- スペース、CamelCase、アンダースコアは使用しない

### コメント言語

- コード内コメント: 日本語

### エラーメッセージ言語

- エラーメッセージ: 英語

### 記事メタデータ

各記事の先頭に以下のメタデータを記載：

```yaml
---
title: "記事タイトル"           # 日本語
emoji: "📝"                     # 1 文字の絵文字
type: "tech"                    # tech または idea
topics: ["javascript", "docker"] # 関連トピック
published: true                 # 公開状態
---
```

## 開発コマンド

### 依存関係のインストール

```bash
pnpm install
```

**重要**: npm や yarn は使用しないでください。必ず pnpm を使用してください。

### 開発サーバーの起動

```bash
pnpm run dev
# http://localhost:8000 でプレビュー可能
```

### 文章校正

```bash
# textlint 実行
pnpm run lint

# 自動修正可能な問題を修正
pnpm run fix
```

## 注意事項

### 認証情報のコミット禁止

- API キーや認証情報は Git にコミットしない
- `.env.local` などの環境変数ファイルで管理する

### ログへの機密情報出力禁止

- ログに個人情報や認証情報を出力しない
- デバッグ情報も公開リポジトリでは慎重に扱う

### 既存ルールの優先

- textlint および markdownlint のルールを優先
- 既存のコーディングスタイルに従う

### 既知の制約

#### パッケージマネージャー

- **必須**: pnpm v10.28.1（`package.json` の `packageManager` フィールドで指定）
- npm や yarn を使用した場合、生成されたロックファイルを削除する

#### Node.js バージョン

- **必須**: v24.13.0（`.node-version` で指定）

#### テキスト処理ワークフロー

1. Markdown ファイルを編集
2. `pnpm run fix` で自動修正
3. `pnpm run lint` で品質確認
4. **手動レビュー**: 自動修正で文章が壊れていないか確認

特に、`pnpm run fix` によって「。」に置き換わるルールがあるため、これによって違和感のある文章となっていないかを確認する必要があります。例えば「例」が「例。」のように末尾に句点が自動付与されることがありますが、この場合は「以下は例。」のように文として成立する形に修正する必要があります。

## Zenn 独自 Markdown 記法

### 画像

```markdown
![alt text](URL =250x)  # サイズ指定
![](URL)
*キャプション*           # キャプション
```

### コードブロック

```markdown
```js:filename.js      # ファイル名表示
const code = "example";
```

```diff js:file.js     # 差分表示
+ 追加行
- 削除行
```
```

### メッセージボックス

```markdown
:::message
通常のメッセージ
:::

:::message alert
警告メッセージ
:::
```

### アコーディオン

```markdown
:::details タイトル
折りたたみ内容
:::
```

### 外部コンテンツ埋め込み

```markdown
@[card](URL)           # リンクカード
@[tweet](URL)          # ツイート
@[youtube](ID)         # YouTube
@[gist](URL)           # Gist
```

## リポジトリ固有

### textlint ルール

170 以上の textlint ルールが設定されています：

- **日本語文法**: い抜き言葉、サ抜き、フィラー（ええと、あの、など）を禁止
- **技術文書品質**: JTF 日本語標準スタイルガイド準拠
- **表記統一**: 専門用語の表記統一（例: "Cloudflare" not "CloudFlare"）
- **スペース**: 日本語と英数字の間に半角スペース必須

### markdownlint ルール

120 以上の markdownlint ルールが設定されています：

- **見出し**: ATX スタイル、段階的インクリメント
- **リスト**: ダッシュスタイル、2 スペースインデント
- **コード**: 言語指定必須
- **フォーマット**: 行末スペース禁止、EOF に単一改行

### 文章スタイル

- **スタイル**: ですます調（丁寧・フォーマル）
- **句読点**: 「、」（カンマ）と「。」（ピリオド）を使用
- **漢字 / ひらがな**: 特定の単語は平仮名を優先（副詞、代名詞など）

### Renovate

このリポジトリでは Renovate bot が依存関係を自動更新します。Renovate が作成したプルリクエストには追加のコミットを行わないでください。

## 参考リンク

- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
- [Conventional Branch](https://conventional-branch.github.io)
- [Zenn CLI ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [Zenn Markdown ガイド](https://zenn.dev/zenn/articles/markdown-guide)
- [textlint](https://textlint.github.io/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/book000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/book000)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
