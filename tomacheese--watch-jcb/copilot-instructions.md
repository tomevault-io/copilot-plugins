## watch-jcb

> このドキュメントは、Gemini CLI 向けのコンテキストと作業方針を定義します。

# GEMINI.md

## 目的
このドキュメントは、Gemini CLI 向けのコンテキストと作業方針を定義します。

## 出力スタイル
- 言語: 日本語
- トーン: 丁寧かつ簡潔
- 形式: Markdown

## 共通ルール
- 会話は日本語で行う。
- コミットメッセージは Conventional Commits に従い、日本語で記載する。
- 日本語と英数字の間には半角スペースを入れる。

## プロジェクト概要
- 目的: JCB カードのキャンペーン情報を監視し、新着情報を Discord に通知する
- 主な機能: キャンペーン一覧の取得、新着判定、通知送信

## コーディング規約
- フォーマット: Prettier
- リンター: ESLint
- コメント言語: 日本語
- エラーメッセージ言語: 英語

## 開発コマンド
```bash
# インストール
pnpm install

# 開発
pnpm dev

# ビルド (型チェック)
pnpm lint:tsc

# テスト
pnpm test

# リンター
pnpm lint

# 自動修正
pnpm fix
```

## 注意事項
- 認証情報 (Webhook URL 等) をコミットしない。
- ログに機密情報を出力しない。
- 既存のプロジェクト構造とスタイルを優先する。

## リポジトリ固有
- スクレイピング対象のサイト構造の変化に注意が必要。
- 文字コード (Shift_JIS (windows-31j)) の扱いに `iconv-lite` を使用している。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomacheese) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
