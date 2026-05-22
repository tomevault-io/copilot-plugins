## sesami-2-mackerel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

SESAMIスマートロックデバイスの状態（バッテリー残量、ロック状態）をMackerelに送信して監視するためのGoアプリケーションです。24時間間隔で定期実行され、メトリクスを自動収集します。

## 開発・実行コマンド

### ローカル実行
```bash
# 環境変数を設定して実行
SESAMI_API_KEY=test_sesami_key SESAMI_DEVICE_UUID=test_device_uuid MACKEREL_API_KEY=test_mackerel_key go run main.go

# または直接実行
go run main.go
```

### Docker Compose実行
```bash
# .envファイルを作成後（compose.yamlを使用）
docker compose up -d
```

### ビルド
```bash
go build -o sesami-2-mackerel .
```

### データベース関連
```bash
# Entスキーマコード生成
go generate ./ent

# Go依存関係の整理
go mod tidy
```

## 必須環境変数

以下の環境変数を`.env`ファイルまたは環境に設定する必要があります：

- `SESAMI_API_KEY`: SESAMI RESTful webAPI Key
- `SESAMI_DEVICE_UUID`: SESAMI Device UUID  
- `MACKEREL_API_KEY`: Mackerel API Key
- `DATABASE_PATH`: SQLiteデータベースファイルのパス（デフォルト: `/data/sesami_2_mackerel.db`）

## アーキテクチャ

- `main.go`: エントリーポイント、設定読み込みとスケジューラー開始
- `internal/config/`: 環境変数からの設定読み込みと検証
- `internal/scheduler/`: cronライブラリを使用した24時間間隔の定期実行
- `internal/sesami/`: SESAMI APIクライアント（デバイス状態取得・履歴取得）
- `internal/mackerel/`: Mackerel APIクライアント（メトリクス送信・サービス管理）
- `internal/database/`: SQLiteデータベースクライアント（Entフレームワーク使用）
- `ent/`: Entによる自動生成されたデータベース層コード（編集不可）
- `ent/schema/`: データベーススキーマ定義（`devicestatus.go`, `devicehistory.go`）
- `docs/`: プロジェクトドキュメント（API仕様、操作履歴タイプ対応表など）
- `.data/`: Docker環境でのデータ永続化ディレクトリ（SQLiteファイル格納）

## データベース設計

**DeviceStatus**（デバイス状態）:
- バッテリー残量、電圧、位置、ステータス、タイムスタンプを記録

**DeviceHistory**（デバイス履歴）:
- イベントタイプ、タイムスタンプ、タグ情報を記録
- 重複データの挿入を防ぐ仕組みを実装済み

## SESAMI API情報

**履歴タイプ対応表**（`docs/sesami.md`参照）:
- Type 1: オープンセンサ施錠
- Type 2: オープンセンサ解錠  
- Type 8: 手動解錠
- Type 90: ドア解錠
- Type 91: ドア施錠

## 現在の実装状況

- **データベース層**: SQLite + Entフレームワークで完全実装
- **SESAMI APIクライアント**: モック実装（HTTP APIコール化が必要）
- **Mackerel APIクライアント**: モック実装（HTTP APIコール化が必要）
- **スケジューラーとconfig**: 実装完了

## 主要な依存関係

- Go 1.24.6
- `entgo.io/ent`: ORM・データベースコード生成
- `github.com/mattn/go-sqlite3`: SQLiteドライバー
- `github.com/robfig/cron/v3`: cronスケジューリング

---
> Source: [cateiru/sesami-2-mackerel](https://github.com/cateiru/sesami-2-mackerel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
