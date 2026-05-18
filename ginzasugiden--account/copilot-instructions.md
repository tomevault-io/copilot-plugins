## account

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

「コンテンツページ生成ちゃん」のアカウント管理用フロントエンド。GitHub Pages でホスティングされる純粋な静的サイト。バックエンドは Google Apps Script（別ディレクトリ `gas/`）。

## Development & Deployment

**ホスティング:** GitHub Pages（カスタムドメイン: account.ginzasugiden.com）

```bash
git push origin main   # デプロイ（GitHub Pages に自動反映）
```

ビルドツール・バンドラー・パッケージマネージャーは一切なし。HTML ファイルをそのまま配信する構成。

## Architecture

### 設定ファイル

| ファイル | 役割 |
|---------|------|
| `CNAME` | カスタムドメイン設定（account.ginzasugiden.com） |

### ページ構成

| ファイル | 役割 |
|---------|------|
| `index.html` | アカウント新規作成フォーム。店舗URL・サービスシークレット・ライセンスキー・店舗名・メールアドレスを収集 |
| `reset-request.html` | パスワードリセット申請フォーム。ユーザーID（店舗URL）とメールアドレスを入力 |
| `reset.html` | パスワードリセット実行フォーム。URL クエリパラメータのトークンを検証、新パスワード（8文字以上）を設定 |

### 技術スタック

- **HTML5 + Vanilla JavaScript**（フレームワークなし）
- **インライン CSS**（外部CSSなし）
- **Google reCAPTCHA v3** でボット対策
- バックエンド通信は GAS エンドポイントへの POST リクエスト

### データフロー

ユーザー入力 → クライアントサイドバリデーション → reCAPTCHA トークン取得 → GAS へ POST → GAS が処理・メール送信

### 主要な規約

- **バリデーション:** ライセンスキー = `SL` + 6桁数字 + `_...`、サービスシークレット = `SP` + 6桁数字 + `_...`、パスワード = 最低8文字
- **GAS エンドポイント:** `/account/create`、`/password/reset-request`、`/password/reset`
- **セキュリティ:** 全フォームに reCAPTCHA v3 必須。期限切れ/無効トークンのエラーハンドリングあり

## GSD固有のルール
- このプロジェクトは「コンテンツページ生成ちゃん」という楽天市場向けツール
- UIは日本語、コードのコメントも日本語OK
- フロントエンド（frontend/）は git push origin main でデプロイ（GitHub Pages）
- GAS（gas/）は clasp push でデプロイ。GitHub には push しない
- セキュリティ: スプレッドシートの構造やGASソースコードは外部公開しない
- .delete API 関連は通常「利用不可」推奨

---
> Source: [ginzasugiden/account](https://github.com/ginzasugiden/account) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
