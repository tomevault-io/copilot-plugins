## update-softwares

> このドキュメントは、Gemini CLI 向けのコンテキストと作業方針を定義します。

# Gemini CLI 作業方針

## 目的

このドキュメントは、Gemini CLI 向けのコンテキストと作業方針を定義します。

## 出力スタイル

- 言語: 日本語
- トーン: プロフェッショナル、明確、簡潔
- 形式: マークダウン

## 共通ルール

- 会話は日本語で行う。
- コミットは Conventional Commits に従う。`<description>` は日本語で記載する。
  - 形式: `<type>: <description>` (例: `feat: ユーザー認証機能を追加`)
- 日本語と英数字の間には半角スペースを入れる。

## プロジェクト概要

- 目的: Linux (apt) と Windows (scoop) の複数マシンに対するパッケージ更新を自動化し、GitHub Issues API と統合して進捗を追跡・報告するツール
- 主な機能:
  - Linux システムの apt パッケージ更新 (dist-upgrade)
  - Windows システムの scoop パッケージ更新
  - GitHub Issues API との統合 (issue ステータスの取得・更新)
  - アトミック更新メカニズム (並行実行時の競合防止)
  - OS の End-of-Life (EOL) 情報の取得と表示
  - ログ機能 (ファイルとコンソール出力)

## コーディング規約

- Python 命名規則に従う (snake_case)
- 関数・クラスには docstring を日本語で記載する (Google style を推奨。必要に応じて reStructuredText または NumPy style も可)
- .editorconfig に従う (UTF-8, LF, 2 スペースインデント)
- コメント言語: 日本語
- エラーメッセージ言語: 英語

## 開発コマンド

```bash
# 依存関係のインストール
pip install -r requirements.txt

# テスト実行用に pytest もインストール
pip install pytest

# テスト実行 (推奨: CI と同様に pytest を使用)
pytest tests/

# Linux 固有テスト
pytest tests/linux

# Windows 固有テスト
pytest tests/windows

# または unittest でも実行可能
python3 -m unittest discover -s tests -p "test_*.py"

# アプリケーション実行 (前提: data/github_token.txt に有効な GitHub トークンを記載)
python3 -m src <ISSUE_NUMBER>
```

## 注意事項

- GitHub トークンは `data/github_token.txt` で管理し、Git にコミットしない。
- ログに個人情報や認証情報を出力しない。
- センシティブな情報をコードに含めない。
- プロジェクトの既存ルールを優先する。
- 既知の制約:
  - GitHub トークン必須 (`data/github_token.txt`)
  - Linux: root 権限必須 (apt-get 操作のため)
  - Windows: scoop インストール必須
  - GitHub Issue 形式要件 (markdown テーブル + コメント)

## リポジトリ固有

- **技術スタック**: Python 3.8-3.13 (CI で検証済み、推奨 3.12 以上), pip, requests==2.32.4, psutil==7.2.1, unittest
- **Renovate 統合**: 外部テンプレート使用 (`github>book000/templates//renovate/base-public`)。Renovate が作成した既存のプルリクエストに対して、追加コミットや更新を行わない。
- **アトミック更新メカニズム**: 並行実行時の競合を防止するため、リトライロジックを実装している。
- **OS EOL 情報表示**: endoflife.date API を利用し、EOL 日が 90 日未満の場合はステータスを赤くマークする。
- **ログシステム**: ファイルログ (DEBUG) + コンソール出力 (INFO)。Linux `/opt/update-softwares/logs`, Windows ユーザープロファイル。
- **Windows scoop アップグレードの特殊性**: 実行中アプリケーションは対話形式で確認 (input() で Y/n)。アプリ停止 → アップグレード → 再起動。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/book000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/book000)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
