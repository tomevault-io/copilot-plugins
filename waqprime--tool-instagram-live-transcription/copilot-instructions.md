## tool-instagram-live-transcription

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

各種プラットフォームの動画・音声をMP3で保存し、AIで文字起こし・要約を行うElectronアプリ。
YouTube、Instagram、X Spaces、Voicy、stand.fm、UTAGEなど、yt-dlp対応の1,800以上のプラットフォームに対応。

## Architecture

### Pythonバックエンド（メインモジュール）

1. **downloader.py** - 動画・音声のダウンロード
   - `yt-dlp`を使用して各種プラットフォームから動画・音声を取得
   - `VideoDownloader`クラスが処理を担当
   - UTAGE/Voicy/stand.fm は専用エクストラクタで対応

2. **audio_converter.py** - 音声抽出・変換
   - `ffmpeg`を使用して動画からMP3音声を抽出
   - HLS/m3u8形式からMP4への変換機能を提供

3. **transcriber.py** - 音声文字起こし
   - 4エンジン対応: faster-whisper, kotoba-whisper, openai-api, local-whisper
   - `KotobaWhisperExternalTranscriber`: PyInstallerビルドでは外部Python経由で実行
   - 3種類の出力: プレーンテキスト、タイムスタンプ付きテキスト、JSON

4. **summarizer.py** - 内容要約
   - ビルトイン: AWS Lambda経由でGemini 2.5 Flash Lite（APIキー不要）
   - Gemini API / OpenAI API（ユーザーのAPIキー使用）

5. **main.py** - メインオーケストレーター
   - 全モジュールを統合し、ワークフロー全体を管理

### 専用エクストラクタ

6. **voicy_extractor.py** - Voicy専用（yt-dlpのVoicyエクストラクタが壊れているため）
   - Seleniumでページを開き、再生ボタンをクリックしてHLS音声URLをキャプチャ
   - voice_id指定時は該当放送の再生ボタンをピンポイントでクリック

7. **standfm_extractor.py** - stand.fm専用
   - HTMLの`__SERVER_STATE__` JSONからM4A音声URLを直接抽出（認証不要）

8. **spotify_extractor.py** - Spotify Podcast専用
   - SpotifyはDRM保護のためyt-dlp非対応。RSSフィード経由で音声を取得
   - Spotify embedページ → 番組名取得 → Apple Podcasts APIでRSSフィード検索 → MP3直接DL
   - APIキー不要、新規依存なし

9. **utage_extractor.py** - UTAGE専用
   - Seleniumを使用してUTAGEページから動画URLを抽出

### Electronフロントエンド（`electron-app/`）

- **main.js** - Electronメインプロセス（自動アップデートチェック含む）
- **renderer.js** - UI制御（エンジン選択、要約プロバイダ選択等）
- **backend/process-bundled.js** - PyInstallerバイナリの実行管理

## Development Setup

```bash
# 仮想環境の作成
python3 -m venv venv
source venv/bin/activate

# 依存関係のインストール
pip install -r requirements.txt

# システム依存パッケージ
brew install ffmpeg  # macOS
```

## Common Commands

### 基本的な使い方
```bash
# link.txtのURLを一括処理
python main.py

# 単一URLを処理
python main.py --url "https://www.youtube.com/watch?v=..."
python main.py --url "https://voicy.jp/channel/XXXX/YYYY"
python main.py --url "https://stand.fm/episodes/XXXX"

# エンジン指定
python main.py --url "URL" --engine faster-whisper   # デフォルト
python main.py --url "URL" --engine kotoba-whisper    # 日本語特化
python main.py --url "URL" --engine openai-api --api-key "sk-..."

# 要約付き
python main.py --url "URL" --summarize                          # ビルトイン
python main.py --url "URL" --summarize --summary-provider gemini --gemini-api-key "AIza..."

# 動画ファイルを保持する
python main.py --url "URL" --keep-video
```

## Input/Output

### 入力
- `link.txt` - 動画・音声URLのリスト（1行1URL）

### 出力（デフォルト: `~/Downloads/InstagramTranscripts/`）
- `*.mp3` - 抽出された音声ファイル
- `*.mp4` - 動画ファイル（`--keep-video`オプション使用時）
- `*_transcript.txt` - 文字起こしテキスト
- `*_transcript_detailed.txt` - タイムスタンプ付き文字起こし
- `*_transcript.json` - 詳細情報（セグメント、信頼度など）
- `*_summary.md` - 内容要約（`--summarize`オプション使用時）

## Technical Notes

- **文字コード**: すべてのファイルI/OでUTF-8エンコーディングを使用
- **文字起こしエンジン**: faster-whisper（デフォルト、large-v3-turbo）が推奨
- **kotoba-whisper**: PyInstallerビルドではtorch/transformersが除外されているため、システムPython経由で実行。初回は自動pipインストール
- **ビルトイン要約**: AWS Lambda + API Gateway経由。APIキーはLambda環境変数に格納（ソースコードに含まない）
- **自動アップデート**: GitHub APIで最新リリースをチェック、全プラットフォーム共通

## Dependencies

- **yt-dlp**: 各種プラットフォームから動画・音声ダウンロード
- **ffmpeg**: 音声抽出・変換（システムパッケージ）
- **faster-whisper**: 高速音声認識（デフォルトエンジン）
- **kotoba-whisper**: 日本語特化音声認識（オプション、システムPython経由）
- **selenium**: Voicy/UTAGE用ブラウザ自動化
- **webdriver-manager**: ChromeDriverの自動インストール
- **openai**: OpenAI API / Gemini API互換クライアント

## Build & Release

### 配布形態
- **macOS / Windows**: Electronアプリ（`electron-app/`）として配布。CLIバイナリ単体ではない
- **CI (GitHub Actions)**: Windows + macOS arm64 を自動ビルド・署名・公証・リリース
- **Intel Mac (x64)**: CIでは対応不可。ローカルで手動ビルドする

### Intel Mac 手動ビルド手順
```bash
# 1. Electronアプリとしてx64ビルド
cd electron-app
npm version X.Y.Z --no-git-tag-version --allow-same-version
APPLE_ID="..." APPLE_APP_SPECIFIC_PASSWORD="..." APPLE_TEAM_ID="..." \
  npm run build:mac-x64

# 2. DMG作成失敗時（macOS 26+ hdiutil日本語ボリューム名バグ）
#    npm run build:mac-x64 でzipは作成されるがDMGがhdiutilエラーで失敗する場合、
#    手動でDMG作成する:
hdiutil create -srcfolder "dist/mac/文字起こしツール.app" \
  -volname "TranscriptionTool" -anyowners -nospotlight \
  -format UDZO -fs APFS "dist/TranscriptionTool-X.Y.Z-x64.dmg"

# 3. リリースにアップロード
gh release upload vX.Y.Z electron-app/dist/TranscriptionTool-*.dmg electron-app/dist/TranscriptionTool-*.zip

# 4. クリーンアップ
rm -rf electron-app/dist dist build
```

### AWS Lambda（ビルトイン要約）
- 関数名: `transcription-summarizer`
- リージョン: ap-northeast-1
- エンドポイント: API Gateway経由
- APIキーはLambda環境変数 `GEMINI_API_KEY` に格納
- スロットリング: 2リクエスト/秒、バースト5

## Known Limitations

- Instagram動画のダウンロードは公開コンテンツに限定される
- 長時間の動画（60分以上）はWhisper処理に時間がかかる場合がある
- kotoba-whisperはシステムにPython 3 + torch + transformersが必要（初回自動インストール）
- Voicy/UTAGEの抽出にはChromeブラウザが必要
- macOS 26以降、hdiutilが日本語ボリューム名（`文字起こしツール`）でDMG作成に失敗する。手動ビルド時はASCIIボリューム名（`TranscriptionTool`）を使用すること

---
> Source: [waqprime/tool-instagram-live-transcription](https://github.com/waqprime/tool-instagram-live-transcription) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
