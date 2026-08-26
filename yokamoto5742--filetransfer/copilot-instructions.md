## filetransfer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

FileTransferは、指定フォルダを監視し、新規作成/移動されたファイルを設定済みの正規表現パターンでリネームして対象ディレクトリへ移動するWindows常駐トレイアプリケーション。移動後にWindows ExplorerのビューをSHChangeNotifyで更新する。

## アーキテクチャ

- `main.py` — エントリーポイント。ロギングを初期化し `TrayApp` を起動。
- `app/tray_app.py` — `TrayApp`: pystrayによるトレイアイコンを管理し、バックグラウンドスレッドで単一の`watchdog.Observer`に監視元を全て登録（`recursive=False`）。監視元ごとに`FileRenameHandler`を1つ生成する。監視開始後、各監視元に既に存在するファイルを処理する。
- `service/file_rename_handler.py` — `FileRenameHandler`: ファイル作成/移動イベントを検知し、書き込み完了を待ってからファイル名に応じた移動先へリネームして移動。`SHChangeNotify`（ctypes経由）でExplorerを更新。設定は読まず、移動先ルールと待機時間をコンストラクタで受け取る。
- `utils/config_manager.py` — `utils/config.ini`の読み込み/保存。PyInstallerでフリーズされた実行ファイルでは`sys._MEIPASS`からパスを解決。
- `utils/log_rotation.py` — `TimedRotatingFileHandler`による日次ログローテーションと古いログの自動削除を設定。

## 依存関係管理

本プロジェクトは **uv** で依存関係を管理する（`pyproject.toml` + `uv.lock`）。`requirements.txt`は存在しない。依存関係の追加・更新は`uv add` / `uv sync`を使用する。

## 型チェック

pyright設定は`pyrightconfig.json`ではなく`pyproject.toml`の`[tool.pyright]`にある。`app`, `service`, `utils`, `tests`を対象とし、`scripts`を除外。

```bash
pyright
```

## ビルド

```bash
python build.py
```

## 実行時設定

`utils/config.ini`に実際の稼働設定が直接コミットされている（テンプレートではない）。
- `[Watch1]`, `[Watch2]`...: 監視元ごとのセクション（番号の昇順で処理）
  - `processing_dir`: 監視対象ディレクトリ
  - `target_dir1`, `target_dir2`...: 移動先ディレクトリ
  - `filename1`, `filename2`...: `target_dirN`に移動するファイル名（カンマ区切り、完全一致、空欄なら全ファイル）
  - `regex1`, `regex2`...: `target_dirN`に移動するファイル名の正規表現（`filenameN`の完全一致に一致しない場合のみ判定、空欄なら無効）
  - `pattern1`, `pattern2`...: `target_dirN`へ移動する際にファイル名末尾へ追加するサフィックス
- `[App]`: `wait_time`
- `[LOGGING]`: `log_retention_days`, `log_directory`, `log_level`, `debug_mode`, `project_name`

振り分けルールは監視元ごとに独立している。起動時に、移動先が監視元と同一のディレクトリを指している場合（無限ループになる）はエラー終了する。

## コーディング規約・コミット規約・テスト

コーディングスタイル、コミットメッセージ形式、レスポンス形式、テスト実行コマンドは `.claude/rules/` 配下に分割されている（`coding-guidelines.md`, `commit.md`, `python-coding.md`, `response-style.md`, `testing.md`）。これらは自動的に読み込まれるため、ここでは重複させない。

## 変更履歴

`docs/CHANGELOG.md`に[Keep a Changelog](https://keepachangelog.com/ja/1.1.0/)形式で記録する。

---
> Source: [yokamoto5742/FileTransfer](https://github.com/yokamoto5742/FileTransfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
