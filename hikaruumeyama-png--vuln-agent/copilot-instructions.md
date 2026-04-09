## vuln-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

SIDfmの脆弱性通知メールを自動解析し、SBOMと突合して担当者へ通知する **Vertex AI Agent Engine** 向けAIエージェント。

Google Sheets(またはBigQuery) / Google Chat を使った運用を前提とし、Google Chat連携・音声/チャットUI連携にも対応。

## 技術スタック

- **言語**: Python 3.12
- **AIフレームワーク**: Google ADK (Agent Development Kit) + Vertex AI Agent Engine
- **モデル**: gemini-2.5-flash
- **インフラ**: Google Cloud (Cloud Run, Cloud Functions, BigQuery, Secret Manager, Cloud Storage)
- **CI/CD**: Cloud Build (`cloudbuild.yaml`)
- **Web UI**: 静的HTML/JS/CSS (Vanilla JS + Lucide Icons)
- **リアルタイム音声**: Gemini Multimodal Live API + FastAPI WebSocket

## ディレクトリ構成

```
agent/                  # メインエージェント (ADK)
├── agent.py              # エージェント定義 (システムプロンプト + ツール登録)
├── requirements.txt      # Agent Engine ランタイム依存
├── .env.example          # 環境変数テンプレート
└── tools/                # ツール群
    ├── sheets_tools.py     # SBOM検索・担当者マッピング (Sheets/BigQuery両対応)
    ├── chat_tools.py       # Google Chat通知 (カード形式)
    ├── history_tools.py    # BigQuery対応履歴記録
    └── a2a_tools.py        # Agent-to-Agent連携

chat_webhook/           # Cloud Functions Google Chat Webhook
├── main.py               # エントリーポイント (handle_chat_event)
└── requirements.txt

live_gateway/           # Cloud Run WebSocket + Gemini Live API
├── app.py                # FastAPI WebSocketサーバー
├── live_api.py           # Gemini Live APIクライアント
├── Dockerfile
└── requirements.txt

web/                    # ブラウザUI (静的配信)
├── index.html
├── app.js
└── style.css

docs/                   # セットアップガイド
setup_cloud.sh          # 初回セットアップ (全自動)
cloudbuild.yaml         # CI/CDパイプライン
```

## コーディング規約

- 日本語コメント・docstring推奨（ユーザー向けメッセージは日本語）
- 型ヒント必須 (`def func(param: str) -> dict[str, Any]:`)
- ログは `logging` モジュール経由（`print` ではなく `logger.info/error`）
- 環境変数は `os.environ.get()` で取得、Secret Managerから注入される前提
- APIサービスはモジュールレベルでキャッシュ（`_SERVICE_CACHE_TTL = 1800`）
- エラーハンドリングは `{"status": "error", "message": str(e)}` 形式で返却

## ツール関数の設計パターン

- 各ツール関数は `dict[str, Any]` を返す
- 成功時: `{"status": "success", ...}` or 具体的なデータ
- 失敗時: `{"status": "error", "message": "..."}`
- ADKの `FunctionTool` でラップされるため、引数にはプリミティブ型 + list を使用
- docstringにArgs/Returns/Exampleを記載（Geminiがツール説明として読む）

## デプロイ

- **初回セットアップ**: `bash setup_cloud.sh`
- **Agent Engine**: `adk deploy agent_engine --project=PROJECT --region=asia-northeast1`
- **CI/CD**: `gcloud builds submit --config cloudbuild.yaml`

## テスト

- `test_agent.sh "メッセージ"` でAgent Engineに直接クエリ
- `test_agent.py` でPython SDKからのストリーミングテスト（`.gitignore` 対象のため各自作成）

## 重要な注意事項

- `agent/.env` は `.gitignore` 済み。Secret Managerから生成される
- `credentials.json` / `token.json` はリポジトリに含めない
- SBOMデータソースは `SBOM_DATA_BACKEND` で sheets/bigquery/auto を切替可能
- Agent Engine デプロイ後のリソース名は `vuln-agent-resource-name` シークレットに自動保存
- Cloud Build の `_AGENT_ENGINE_RETENTION` で古いエンジンの自動削除数を制御

## 言語設定
- 常に日本語で回答すること。英語での回答は禁止。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hikaruumeyama-png)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hikaruumeyama-png)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
