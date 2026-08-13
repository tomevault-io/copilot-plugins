## claude-code-perlnavigator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Claude Code に Perl 言語サポートを提供するプラグイン。PerlNavigator LSP サーバーを stdio トランスポートで統合し、
Perl ファイル (.pl, .pm, .t, .cgi, .psgi) の診断・補完・ナビゲーション機能を提供する。

## アーキテクチャ

```
Claude Code → Plugin System → .lsp.json → PerlNavigator LSP (perlnavigator --stdio)
                                ↑
                          hooks.json (SessionStart) → check-perlnavigator.sh
```

- `.lsp.json`: LSP サーバーの接続設定（コマンド、対応拡張子、トランスポート方式）
- `.claude-plugin/plugin.json`: プラグインメタデータ（名前、バージョン、キーワード）
- `.claude-plugin/marketplace.json`: マーケットプレイス配布用マニフェスト
- `hooks/hooks.json`: SessionStart フックで `check-perlnavigator.sh` を実行（タイムアウト10秒）
- `scripts/check-perlnavigator.sh`: perlnavigator のインストール・更新状態を確認しユーザーに通知

## ビルド・テスト

ビルドやテストの仕組みはない。設定ファイルとシェルスクリプトのみで構成されるプラグインのため。

## 前提条件

```bash
npm install -g perlnavigator-server
```

---
> Source: [yuji-hatakeyama/claude-code-perlnavigator](https://github.com/yuji-hatakeyama/claude-code-perlnavigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
