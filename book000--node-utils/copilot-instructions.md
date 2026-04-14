## node-utils

> このドキュメントは、Gemini CLI がこのプロジェクトで作業する際のコンテキストと方針を定義します。

# Gemini CLI 向けコンテキストと作業方針

## 目的

このドキュメントは、Gemini CLI がこのプロジェクトで作業する際のコンテキストと方針を定義します。

## 出力スタイル

- **言語**: 日本語で回答する
- **トーン**: 簡潔で明確な表現を使用する
- **形式**: 必要に応じてコードブロック、リスト、表を使用する

## 共通ルール

- 会話は日本語で行う
- コミットメッセージは [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) に従う。`<description>` は日本語で記載する
- ブランチ命名は [Conventional Branch](https://conventional-branch.github.io) に従う。`<type>` は短縮形 (feat, fix, docs など) を使用する
- 日本語と英数字の間には半角スペースを挿入する

## プロジェクト概要

- プロジェクト名: @book000/node-utils
- 目的: TypeScript による個人向けユーティリティライブラリの開発
- 主な機能:
  - **Logger**: Winston ベースのロガーラッパークラス (日本のタイムゾーン対応、日次ローテーション、NDJSON 対応)
  - **ConfigFramework**: JSONC 形式の設定ファイル管理フレームワーク (バリデーション機能、環境変数対応)
  - **Discord**: Discord Bot / Webhook を使用したメッセージ送信ユーティリティ (埋め込み、ファイル送信、リンクボタン対応)

## 技術スタック

- 言語: TypeScript 5.9.3
- ランタイム: Node.js v24.13.0
- パッケージマネージャー: pnpm 10.28.1 (必須)
- ビルドターゲット: ES2020 (CommonJS モジュール形式)
- テストフレームワーク: Jest 30.2.0 + ts-jest 29.4.6

## コーディング規約

- **フォーマット**: Prettier 3.8.1 (セミコロン非使用、シングルクォート、インデント 2 スペース)
- **リント**: ESLint 9.39.2 + @book000/eslint-config 1.12.40
- **TypeScript**: 厳密モード有効、`skipLibCheck` での回避は禁止
- **コメント**: 関数・インターフェースには JSDoc コメントを日本語で記載
- **エラーメッセージ**: 英語で記載
- **命名規則**: クラス名 (PascalCase)、関数・変数名 (camelCase)

## 開発コマンド

```bash
# 依存関係のインストール (必須: pnpm を使用)
pnpm install

# TypeScript ファイルを実行
pnpm start

# ファイル変更監視モードで実行
pnpm dev

# Examples を実行
pnpm example

# ビルド (ctix 生成 + TypeScript コンパイル)
pnpm build

# クリーン (dist と output ディレクトリを削除)
pnpm clean

# テスト実行 (カバレッジ報告付き)
pnpm test

# リント実行 (Prettier + TypeScript + ESLint)
pnpm lint

# 自動修正 (Prettier + ESLint)
pnpm fix
```

## 注意事項

- **認証情報のコミット禁止**: API キーや認証情報を Git にコミットしない
- **ログへの機密情報出力禁止**: ログに個人情報や認証情報を出力しない
- **既存ルールの優先**: プロジェクトの既存のフォーマット・命名規則に従う
- **既知の制約**:
  - ctix により `src/index.ts` が自動生成される。手動編集は禁止
  - `src/examples/**` は Index ファイル生成対象外
  - Renovate による自動依存更新が有効。Renovate が作成した既存の PR に対して追加コミットや更新を行わない

## リポジトリ固有

- `tsconfig.build.json` で examples と tests を除外してビルドする
- 発行前に `npm run lint` が自動実行される (`prepublishOnly` スクリプト)
- npm install 後に `npm run build` が自動実行される (`prepare` スクリプト)
- リポジトリは `git@github.com:book000/node-utils.git` (upstream) と `git@github.com:akubiusa/node-utils.git` (origin) の 2 つのリモートがある

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/book000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/book000)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
