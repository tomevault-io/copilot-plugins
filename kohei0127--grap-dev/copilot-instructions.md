## grap-dev

> このファイルは Claude Code がこのプロジェクトで作業するときに読み込むガイダンスです。

# CLAUDE.md

このファイルは Claude Code がこのプロジェクトで作業するときに読み込むガイダンスです。

## このプロジェクトについて

`C:\claude_neko` は Claude Code CLI を使うための作業用ワークスペースです。
特定の言語やフレームワークに固定されていない汎用フォルダとして運用します。

## 環境

- OS: Windows 11
- シェル: PowerShell が主。POSIX が必要なときは Bash（Git Bash）を使う
- Claude Code / Git はインストール済み（Node.js は PATH に無い場合あり）

## 作業上のルール

- 回答・コミットメッセージ・ドキュメントは日本語で書く
- ファイルパスを示すときは絶対パス（例: `C:\claude_neko\README.md`）を使う
- 破壊的な操作（ファイル削除・上書き・force push など）を行う前に確認する
- コミットはユーザーから明示的に依頼されたときだけ行う

## 便利なコマンド

| コマンド | 説明 |
|---------|------|
| `claude` | 対話モードを開始 |
| `claude "タスク"` | 1回だけタスクを実行 |
| `claude -c` | 直前の会話を続ける |
| `/init` | プロジェクトを解析して CLAUDE.md を更新 |
| `/help` | ヘルプ表示 |

---
> Source: [kohei0127/grap_dev](https://github.com/kohei0127/grap_dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
