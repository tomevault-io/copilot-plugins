## pod-tenuki

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 開発環境

- Python 3.10-3.13 を使用（pyproject.toml で指定）
- 依存関係管理には `uv` を推奨（`pip3` でも可能）
- 仮想環境 (.venv) を使用
- コマンド実行時は常に `python3` を使用（`python` ではない）
- 外部依存: `ffmpeg`（音声処理に必要）

### セットアップ

```bash
# uv を使用する場合
uv sync

# pip を使用する場合
source .venv/bin/activate
pip3 install -e .
```

## プロジェクト構造

このプロジェクトは、ポッドキャスト音声ファイルを処理するための CLI ツールで、以下の3つの主要な処理パイプラインを持つ:

1. **音声変換** (`audio_converter/`): Auphonic API を使用して音声を変換、または複数の WAV ファイルを連結
2. **文字起こし** (`transcriber/`): Google Cloud Speech-to-Text API を使用して音声をテキスト化
3. **要約生成** (`summarizer/`): OpenAI API (GPT-4o) を使用してタイトルと説明文を生成

### モジュール構成

- `main.py`: メイン CLI エントリーポイント（全パイプラインを実行）
- `cli/`: 各処理の個別 CLI ツール
  - `convert.py`: 音声変換のみ
  - `transcribe.py`: 文字起こしのみ
  - `summarize.py`: 要約生成のみ
- `audio_converter/`: 音声処理
  - `auphonic.py`: Auphonic API クライアント
  - `wav_concat.py`: WAV ファイル連結（ffmpeg を使用）
- `transcriber/`: 文字起こし
  - `google_speech.py`: Google Cloud Speech-to-Text API クライアント（長時間音声対応）
  - `gemini_transcriber.py`: Gemini API クライアント（未使用）
- `summarizer/`: 要約生成
  - `openai_summarizer.py`: OpenAI API クライアント（GPT-4o を使用）
- `utils/`: 共通ユーティリティ
  - `config.py`: 環境変数と設定の検証
  - `cost_tracker.py`: API 使用コストの追跡
  - `logger.py`: ロギング設定

## コマンド

### メイン CLI（全パイプライン実行）

```bash
# 基本的な使い方
uv run pod-tenuki /path/to/audio.mp3

# または pip でインストールした場合
pod-tenuki /path/to/audio.mp3

# 複数の WAV ファイルを連結して処理
uv run pod-tenuki file1.wav file2.wav file3.wav

# 特定の処理をスキップ
uv run pod-tenuki --skip-conversion audio.mp3  # 変換をスキップ
uv run pod-tenuki --skip-transcription audio.mp3  # 文字起こしをスキップ
uv run pod-tenuki --skip-summarization audio.mp3  # 要約をスキップ

# 出力先ディレクトリを指定（デフォルト: ./output）
uv run pod-tenuki --output-dir /path/to/output audio.mp3
```

### 個別 CLI ツール

```bash
# 音声変換のみ
uv run pod-tenuki-convert audio.mp3

# 文字起こしのみ
uv run pod-tenuki-transcribe audio.mp3

# 要約生成のみ（既存の .txt ファイルから）
uv run pod-tenuki-summarize audio.txt
```

## API キー設定

`.env` ファイルに以下を設定:

```
AUPHONIC_API_KEY=your_auphonic_api_key
OPENAI_API_KEY=your_openai_api_key
GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
GOOGLE_CLOUD_PROJECT=your-gcp-project-id
GOOGLE_STORAGE_BUCKET=your-bucket-name
```

Google Cloud Speech-to-Text API を使用するには、以下の権限を持つサービスアカウントが必要:
- Speech to Text ユーザー
- Storage オブジェクト管理者（または objects.create/get/delete 権限）

## 重要な仕様

- **文字起こし**: 長時間音声（最大8時間）に対応。Google Cloud Storage に一時アップロードして処理
- **要約生成**: GPT-4o を使用（約 $0.10/要約）
- **日本語要約**: 必ず「本人曰く、全ての話はフィクションであることを留意してお楽しみください。」を含める
- **コスト追跡**: 処理完了時に API 使用コストを表示
- **出力ファイル**: デフォルトで `./output` ディレクトリに保存
  - 文字起こし: `{basename}.txt`
  - 要約: `{basename}.summary.md`

## 開発時の注意点

- `src/pod_tenuki/summarizer/openai_summarizer.py`: GPT-4o を使用しているため、モデル変更時はコスト追跡も更新する
- Google Cloud Storage のライフサイクルポリシー（1日で自動削除）を設定推奨
- 文字起こしは長時間かかる（1時間の音声で 20-30 分程度）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uzulla)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/uzulla)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
