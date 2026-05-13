## tsumiki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

TsumikiはAI駆動開発フレームワークです。Claude Code Plugin経由でインストールされ、要件定義から実装までのAI支援開発プロセスを提供します。

このリポジトリには以下が含まれています：
- **`commands/`**: Claude Codeスラッシュコマンド用のテンプレートファイル（`.md`と`.sh`）
- **`agents/`**: Claude Codeエージェント用の定義ファイル（`.md`）
- **`.claude-plugin/`**: Claude Code Plugin設定ファイル

## 開発コマンド

```bash
# 開発環境
pnpm install                # 依存関係のインストール

# コード品質
pnpm secretlint             # シークレット情報の検査

# pre-commitフック
pnpm prepare                # simple-git-hooksのセットアップ
```

## プロジェクト構造

- **`commands/`**: TsumikiのAI開発フレームワーク用Claude Codeコマンドテンプレート（`.md`と`.sh`ファイル）
- **`agents/`**: Claude Codeエージェント定義（`.md`ファイル）
- **`.claude-plugin/`**: Claude Code Plugin設定（marketplace.json, plugin.json）
- **`book/`**: 開発ガイドとドキュメント

## 技術スタック

- **Security**: secretlint（機密情報検査）
- **Package Manager**: pnpm
- **Distribution**: Claude Code Plugin Marketplace

## インストール方法

ユーザーは以下のコマンドでTsumikiをインストールします：

```bash
/plugin marketplace add https://github.com/classmethod/tsumiki.git
/plugin install tsumiki@tsumiki 
```

Claude Code Pluginが自動的に：
1. リポジトリから`commands/`と`agents/`のファイルを読み込み
2. `.claude-plugin/plugin.json`の設定に従ってコマンドとエージェントを登録
3. `/tsumiki:` プレフィックス付きでコマンドを使用可能にする

## 品質管理

Pre-commitフックで以下が自動実行されます：
- `pnpm secretlint`: 機密情報のチェック

コマンドファイル（`.md`）やエージェント定義（`.md`）を修正する際は、機密情報が含まれていないことを確認してからコミットしてください。

---
> Source: [classmethod/tsumiki](https://github.com/classmethod/tsumiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
