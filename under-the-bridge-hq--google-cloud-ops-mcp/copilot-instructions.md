## google-cloud-ops-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

GCP Cloud Logging / Cloud Monitoring を AI（Claude Code等）から安全に使うための MCP サーバー。
Go言語で実装し、単一バイナリとして配布可能。

## 開発コマンド

Taskfileを使用。`task --list`で全コマンド確認。

```bash
# ビルド（vet実行後にビルド）
task build

# テスト
task test              # 全テスト
task test-short        # 短時間テストのみ
task test-coverage     # カバレッジレポート生成

# コード品質
task fmt               # フォーマット
task vet               # go vet
task lint              # golangci-lint
task fix               # import整理 + フォーマット
task check             # 全品質チェック（fmt, vet, lint, test）

# MCP動作確認
task mcp-init          # initialize テスト
task mcp-tools         # tools/list テスト
```

## コード修正後の必須手順

```bash
# import追加後は必ず実行
task fix

# コミット前に実行
task check
```

## アーキテクチャ

```
├── main.go                  # エントリポイント
├── internal/
│   ├── mcp/server.go        # MCP JSON-RPC処理（stdio）
│   ├── logging/client.go    # Cloud Logging API
│   └── monitoring/client.go # Cloud Monitoring API
├── config.yaml.example      # 設定例
└── Taskfile.yml             # タスク定義
```

### 設計原則

- **thin wrapper**: API呼び出しと最小整形のみ。推論・分析はAIに委譲
- **ガードレール**: allowlist、時間範囲制限、件数制限を実装（PoC以降）
- **出力の安定**: JSON構造を固定

## MCP Tools

| ツール | 目的 |
|--------|------|
| `logging.query` | Logs Explorer相当の検索 |
| `logging.top_errors` | エラー上位を集計（PoC） |
| `monitoring.query_time_series` | メトリクス時系列取得 |
| `monitoring.list_metric_descriptors` | 利用可能メトリクス探索（PoC） |

詳細スキーマは `docs/design/concept.md` を参照。

## GCP認証

ADC（Application Default Credentials）を使用:

```bash
gcloud auth application-default login
```

必要な権限:
- `roles/logging.viewer`
- `roles/monitoring.viewer`

## コードスタイル

- ドキュメント・コメント: 日本語
- Gitコミットメッセージ: 日本語
- 過剰エンジニアリングの回避
  - 必要最小限の変更のみ実装
  - 3行程度の類似コードは抽象化しない
  - 発生し得ないシナリオのエラーハンドリング不要

---
> Source: [under-the-bridge-hq/google-cloud-ops-mcp](https://github.com/under-the-bridge-hq/google-cloud-ops-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
