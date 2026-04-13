## code-typing-trainer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

code-typing-trainer (ctt) - Git リポジトリから識別子を抽出し、Monkeytype 互換のタイピング練習問題セットを生成する CLI ツール。開発者が実際のコードベースの語彙でタイピング練習できる。

## ビルドコマンド

```bash
# 依存関係のインストール
npm install

# ビルド (tsup)
npm run build

# テスト実行 (vitest)
npm test

# 単一テストファイルの実行
npm test -- src/core/normalizer.test.ts

# 型チェック
npm run typecheck

# リント
npm run lint
```

## アーキテクチャ

```
CLI Layer (gunshi)
    ↓
Core Domain Layer (Analyzer, Normalizer, Exporter)
    ↓
Plugin System (TypeScript/Python/Rust parsers)
    ↓
Infrastructure Layer (SQLite via better-sqlite3, Git via simple-git)
```

### ディレクトリ構成

- `src/cli/` - gunshi フレームワークを使用した CLI コマンド
- `src/core/` - コアドメインロジック（analyzer, normalizer, exporter）
- `src/plugins/` - 言語パーサープラグイン（TypeScript パーサーは TS Compiler API を使用）
- `src/infra/` - データベース（SQLite）と Git 連携
- `src/config/` - 設定の読み込みとバリデーション（Zod スキーマ）
- `src/types/` - 共通型定義

### CLI コマンド

- `ctt init <path>` - リポジトリを登録
- `ctt scan [repo]` - コードベースからトークンを抽出
- `ctt status [repo]` - 更新を確認（コミットハッシュを比較）
- `ctt export [repo]` - 問題セットを出力（JSON/テキスト形式）
- `ctt stats [repo]` - 練習統計を表示
- `ctt repos list|remove` - 登録リポジトリを管理

## 技術スタック

- **言語**: TypeScript
- **CLI**: gunshi
- **データベース**: better-sqlite3 (SQLite)
- **Git**: simple-git
- **パーサー**: TypeScript Compiler API
- **バリデーション**: Zod
- **テスト**: vitest
- **ビルド**: tsup

## 主要な概念

### トークンの正規化

すべての識別子は正規化形式（snake_case）に変換され、バリアントをグループ化する：

| 入力 | 正規化形式 | セグメント | ケーシング |
|------|-----------|----------|-----------|
| `getUserProfile` | `get_user_profile` | `["get","user","profile"]` | camel |
| `GetUserProfile` | `get_user_profile` | `["get","user","profile"]` | pascal |
| `get_user_profile` | `get_user_profile` | `["get","user","profile"]` | snake |
| `MAX_RETRY_COUNT` | `max_retry_count` | `["max","retry","count"]` | screaming_snake |

### データモデル

- **Repository** - 最後にスキャンしたコミットハッシュを持つ登録リポジトリ
- **Token** - 抽出された識別子（raw 形式、正規化形式、ケーシング、種別、頻度）
- **TypingStats** - 正規化キーごとの練習統計
- **VariantStats** - raw バリアントごとの練習統計

### 設定ファイル

リポジトリルートに任意で配置する `ctt.config.json` でフィルタリングルールを設定：
- `filter.minLength` / `maxLength` - 識別子の長さ制限
- `filter.excludePatterns` - 除外する正規表現パターン
- `filter.excludeKinds` - 除外するトークン種別（function, variable など）
- `scan.exclude` - スキャンから除外する glob パターン

## Phase 1 MVP スコープ

現在実装中: init, scan, status, export コマンド（TypeScript パーサーのみ）。統計機能と追加パーサー（Python, Rust）は後続フェーズで予定。

## ドキュメンテーション方針

各ディレクトリには `CLAUDE.md` を配置し、以下の内容を記載する：

1. **階層と位置づけ** - トップレベルからの区分け、なぜそのディレクトリがその場所にあるか
2. **格納内容** - そのディレクトリに何を置くか、ファイルの種類や役割
3. **コーディング規約** - そのディレクトリ内のファイルを操作する際に守るべきルール

これにより、Claude Code がコードベースを理解し、一貫性のあるコードを生成できるようにする。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shiroinock)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shiroinock)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
