## github-changelog-translator

> このドキュメントは、Gemini CLI がこのプロジェクトで作業を行う際のコンテキストと作業方針を定義します。

# Gemini CLI 向けコンテキストと作業方針

## 目的

このドキュメントは、Gemini CLI がこのプロジェクトで作業を行う際のコンテキストと作業方針を定義します。

## 出力スタイル

- **言語**: 日本語
- **トーン**: 専門的かつ明確
- **形式**: マークダウン形式で構造化された出力

## 共通ルール

- **会話言語**: 日本語
- **コミット規約**: [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従う
  - `<type>(<scope>): <description>` 形式
  - `<description>` は日本語で記載
  - 例: `feat: ユーザー認証機能を追加`
- **ブランチ命名**: [Conventional Branch](https://conventional-branch.github.io) に従う
  - `<type>/<description>` 形式
  - `<type>` は短縮形（feat, fix）を使用
  - 例: `feat/add-user-auth`
- **日本語と英数字の間**: 半角スペースを挿入

## プロジェクト概要

- **目的**: GitHub Changelog の RSS フィードを翻訳して配信する
- **主な機能**:
  - GitHub Blog Changelog の RSS フィード取得
  - Google Apps Script 経由でのタイトルと本文の翻訳（英語→日本語）
  - 翻訳済み RSS フィードの生成と GitHub Pages での公開
- **配信先**: https://book000.github.io/github-changelog-translator/changelog.xml
- **技術スタック**:
  - 言語: TypeScript 5.9.3
  - ランタイム: Node.js 24.13.0
  - パッケージマネージャー: pnpm 10.28.1
  - HTTP クライアント: axios 1.13.2
  - XML パーサー: fast-xml-parser 5.3.3

## コーディング規約

- **フォーマッター**: Prettier 3.8.1
  - 行幅: 80 文字
  - インデント: 2 スペース
  - 改行コード: LF
  - クォート: シングルクォート
  - トレイリングカンマ: es5（ES5 互換位置で有効）
- **リンター**: ESLint 9.39.2 + @book000/eslint-config 1.12.40
- **TypeScript**: strict モード有効
- **コメント**: 日本語で記載
- **エラーメッセージ**: 英語で記載
- **JSDoc**: 関数・インターフェースに日本語で記載

## 開発コマンド

```bash
# 依存関係のインストール
pnpm install

# RSS フィードの取得と翻訳を実行
pnpm start

# 開発モード（ファイル変更を監視して自動再実行）
pnpm dev

# Lint チェック（Prettier, ESLint, TypeScript）
pnpm lint

# Lint 自動修正（Prettier, ESLint）
pnpm fix

# Prettier チェックのみ
pnpm lint:prettier

# ESLint チェックのみ
pnpm lint:eslint

# TypeScript 型チェックのみ
pnpm lint:tsc

# Prettier 自動修正
pnpm fix:prettier

# ESLint 自動修正
pnpm fix:eslint
```

## 注意事項

### セキュリティ / 機密情報

- **認証情報のコミット禁止**: `GAS_URL` などの環境変数を含むファイルを Git にコミットしない
- **ログへの機密情報出力禁止**: ログに個人情報や認証情報を出力しない
- **環境変数管理**: GitHub Actions の Secrets で環境変数を管理する

### 既存ルールの優先

- このプロジェクトの既存のコーディング規約とパターンに従う
- 既存のフォーマット設定（Prettier, ESLint）を尊重する

### 既知の制約

- **テストフレームワーク未導入**: 品質保証は TypeScript strict モード、ESLint、Prettier、GitHub Actions CI/CD で実施
- **単一ファイル構成**: `src/main.ts` のみのため、モジュール分割は不要
- **外部依存**: Google Apps Script エンドポイントを呼び出すため、`GAS_URL` 環境変数が必須

## リポジトリ固有

- このプロジェクトは GitHub Pages にデプロイされる
- 翻訳処理は外部の Google Apps Script エンドポイントを呼び出す
- 環境変数:
  - `GAS_URL` (必須): Google Apps Script のエンドポイント URL
  - `GITHUB_CHANGELOG_URL` (オプション, デフォルト: https://github.blog/changelog/feed/)
  - `BEFORE_LANGUAGE` (オプション, デフォルト: 'en')
  - `AFTER_LANGUAGE` (オプション, デフォルト: 'ja')
  - `OUTPUT_CHANGELOG_PATH` (オプション, デフォルト: 'output/changelog.xml')
- GitHub Actions による日次自動実行（UTC 0:00）がスケジュールされている
- Renovate による依存関係の自動更新が設定されている。Renovate が作成した PR に追加コミットや更新を行ってはならない
- TypeScript プロジェクトにおいて、`skipLibCheck` を有効にして回避することは絶対にしてはならない

## Gemini の役割

Gemini CLI は、以下の用途で使用されます:

- **外部仕様の確認**: SaaS 仕様、言語・ランタイムのバージョン差、料金・制限・クォータといった最新情報の確認
- **一次情報の検証**: 外部 API やライブラリの最新仕様の調査
- **外部前提条件の検証**: プロジェクトが依存する外部サービスの制約や要件の確認

Gemini CLI を使用する際は、以下を心がけてください:

- 最新の情報を提供する
- 出典を明示する
- 不確実な情報は推測であることを明記する

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/book000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/book000)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
