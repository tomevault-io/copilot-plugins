## nullevi03

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

「ナルエビちゃん三世」は、Claude Code を Telegram Bot 経由で常時稼働させるための極小ラッパー。リポジトリ本体には Bot ロジックは存在せず、`claude` CLI を `claude-plugins-official` の `plugin:telegram` チャンネルに繋いで起動・再起動するだけのシェルスクリプトで構成されている。

## 起動方法

```sh
TELEGRAM_BOT_TOKEN=... TELEGRAM_CHAT_ID=... ./boot.sh
```

- `boot.sh` は `claude --dangerously-skip-permissions --channels plugin:telegram@claude-plugins-official -c` を無限ループで実行し、終了したら 5 秒待って再起動する。
- 初回起動と再起動の各イベントを Telegram にプッシュ通知する (`notify_telegram` 関数)。
- `-c` フラグで前回セッションを継続するため、会話状態は Claude Code 側のセッション履歴に依存する。

## 必須の前提

- Claude Code が CLI として導入されていること (Max プラン等の課金が前提と README に記載)。
- `claude-plugins-official` 配下の Telegram プラグインが設定済みであること。Bot 作成は BotFather で行う。
- 環境変数 `TELEGRAM_BOT_TOKEN` と `TELEGRAM_CHAT_ID` を実際の値に置換する (boot.sh 内のデフォルト値はダミー)。

## 編集時の注意

- スクリプトは POSIX sh で書かれている (`#!/bin/sh`)。bash 固有構文を持ち込まないこと。
- 認証情報は boot.sh にハードコードせず、必ず環境変数経由で渡す前提を崩さない。
- `--dangerously-skip-permissions` を外す変更は挙動を大きく変えるため、ユーザー確認を取ること。

---
> Source: [GOROman/nullevi03](https://github.com/GOROman/nullevi03) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
