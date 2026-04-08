## iot-agent

> このプロジェクトは、FastAPIベースのIoT管理サーバーと、それに対応するエッジデバイス（Jetson, Raspberry Pi, Picoなど）の制御システムです。

# IoT Agent プロジェクト

## プロジェクト概要
このプロジェクトは、FastAPIベースのIoT管理サーバーと、それに対応するエッジデバイス（Jetson, Raspberry Pi, Picoなど）の制御システムです。
ユーザーはWebインターフェース（チャット）を通じて自然言語で指示を出し、LLM（OpenAI, Anthropic, Geminiなど）がそれを解釈して適切なデバイスコマンドを生成・実行します。

## 技術スタック
- **言語**: Python 3.11+, JavaScript (Frontend)
- **フレームワーク**: FastAPI (Backend)
- **AI/LLM**: OpenAI API, Anthropic API, Google Gemini API
- **エッジデバイス**: Python (Jetson, Raspberry Pi), MicroPython (Pico W)
- **コンテナ化**: Docker, Docker Compose

## セットアップと実行

### 前提条件
- Python 3.11以上
- 必要なAPIキー（OpenAI, Anthropic, Geminiなど）

### ローカルでの実行
1. 依存関係のインストール:
   ```bash
   pip install -r requirements.txt
   ```
2. 環境変数の設定:
   `secrets.env` または環境変数で API キーを設定してください。
   ```bash
   export OPENAI_API_KEY="sk-..."
   ```
3. サーバー起動:
   ```bash
   uvicorn app:app --host=0.0.0.0 --port=5006 --reload
   ```

### Dockerでの実行
```bash
docker-compose up --build
```

## ディレクトリ構成

- **根ディレクトリ**:
  - `app.py`: アプリケーションのエントリーポイント。FastAPIサーバー定義、ルートハンドリング。
  - `model_selection.py`: LLMモデルの選択ロジック。
  - `frontend/`: Vite + React SPA（`src/`がソース、`dist/`がビルド成果物）。
  - `requirements.txt`: Python依存パッケージリスト。
  - `Dockerfile`, `docker-compose.yml`: コンテナ化設定。

- **iot_agent/**:
  - アプリケーションのコアロジックを含むパッケージ。
  - `llm.py`: LLMプロバイダーとの通信処理。
  - `execution.py`: デバイスコマンドの実行管理。
  - `state.py`: デバイス、ジョブ、結果のインメモリ状態管理。
  - `device_utils.py`: デバイス関連のユーティリティ。
  - `config.py`: 設定値の読み込み。

- **edge_device_code/**:
  - エッジデバイス用のクライアントコード例。
  - `jetson/`: NVIDIA Jetson用。
  - `raspberrypi4/`: Raspberry Pi 4用。
  - `raspberrypi-pico/`: Raspberry Pi Pico W用 (MicroPython)。

## 主要ファイル詳細

### `app.py`
メインサーバー。以下の機能を担当します：
- 認証（セッションベース）
- 静的ファイル（UI）の配信
- APIエンドポイントの提供（チャット、デバイス登録、ジョブ管理）
- WebSocket/ポーリングによる状態更新

### `iot_agent/llm.py`
LLMとの対話を抽象化しています。
- `_call_llm_and_parse`: プロンプトを送信し、JSON形式のコマンドや応答をパースします。
- 各種プロバイダー（OpenAI, Anthropic）のクライアント初期化を含みます。

### `edge_device_code/*/`
各デバイス用のスクリプトは、サーバーの `/api/devices` エンドポイントに対してポーリングを行い、
自分宛てのジョブ (`/jobs/next`) を取得・実行し、結果 (`/jobs/result`) を返します。

## 開発ガイドライン
- **状態管理**: 現状、データベースは使用せず、すべての状態（デバイス、ジョブ）はメモリ上に保持されます。サーバー再起動でリセットされる点に注意してください。
- **認証**: 簡易的なパスワード認証が実装されています。デフォルトパスワードは `app.py` 内の定数または環境変数で設定されます。
- **日本語対応**: ユーザーは日本語環境を好むため、UIやLLMのプロンプト、コメントなどは日本語または日本語対応を意識してください。

## Critical Instruction / 重要指示
どれだけ時間とコストをかけてもよいので、確実に目標が達成されるようにしてほしい。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kota-kawa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kota-kawa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
