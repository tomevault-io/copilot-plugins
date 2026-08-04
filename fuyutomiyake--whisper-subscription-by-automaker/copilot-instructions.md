## whisper-subscription-by-automaker

> 音声録音とOpenAI Whisper APIを使った文字起こし機能を提供するプロジェクトです。

# Whisper Launcher プロジェクト

## 概要
音声録音とOpenAI Whisper APIを使った文字起こし機能を提供するプロジェクトです。
録音した音声を自動的に文字起こしし、結果をクリップボードにコピーします。

## セットアップ

### 必要な依存関係
- Python 3.11以上
- uv (Pythonパッケージマネージャー)
- sox (音声録音ツール)
- OpenAI API キー

### インストール手順
```bash
# 依存関係のインストール
brew install sox
curl -LsSf https://astral.sh/uv/install.sh | sh

# Python依存関係のインストール
uv sync
```

### 設定ファイル
- `hack/run.sh`: OpenAI API キーの設定ファイル
- `hack/record_and_transcribe.sh`: 録音と文字起こしを実行するメインスクリプト

## 実行方法

### コマンドライン実行
```bash
# プロジェクトディレクトリに移動
cd /Users/fm/Downloads/whisper-launcher-main

# 録音と文字起こしを実行
bash hack/record_and_transcribe.sh
```

### Automatorアプリとして実行
1. Automatorで「アプリケーション」として作成
2. 「シェルスクリプトを実行」アクションを追加
3. スクリプトパス: `/Users/fm/Downloads/whisper-launcher-main/hack/record_and_transcribe.sh`

## 動作原理
1. soxコマンドで音声録音（3秒間の無音で自動終了）
2. 録音ファイル（audio.wav）をOpenAI Whisper APIで文字起こし
3. 結果をpyperclipでクリップボードにコピー

## ファイル構成
- `main.py`: メインの文字起こし処理
- `hack/run.sh`: 環境変数設定スクリプト
- `hack/record_and_transcribe.sh`: 録音と文字起こしの統合スクリプト
- `pyproject.toml`: Python依存関係の定義

## トラブルシューティング

### よくある問題
1. **スクリプトが終了しない**: 3秒間完全に無音にする
2. **マイクアクセスエラー**: システム環境設定でマイク許可を確認
3. **APIエラー**: `hack/run.sh`のAPIキー設定を確認

### デバッグコマンド
```bash
# ログファイルの確認
cat sox_test.log

# APIキーの確認
source hack/run.sh && echo $OPEN_AI_API_KEY

# 録音ファイルの確認
ls -la audio.wav
```

---
> Source: [FuyutoMiyake/Whisper-subscription-by-Automaker-](https://github.com/FuyutoMiyake/Whisper-subscription-by-Automaker-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
