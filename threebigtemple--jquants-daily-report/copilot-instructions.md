## jquants-daily-report

> 本プロジェクトでは、以下のサブエージェントを活用して並列開発を効率化する。

# サブエージェント設定 - J-Quants Daily Report

## 概要

本プロジェクトでは、以下のサブエージェントを活用して並列開発を効率化する。

---

## Agent 1: API Client Agent

### 役割
J-Quants APIクライアントの実装を担当

### 担当ファイル
- `src/jquants_report/api/client.py`
- `src/jquants_report/api/auth.py`
- `src/jquants_report/api/endpoints.py`
- `tests/test_api.py`

### プロンプト
```
あなたはJ-Quants APIクライアントの実装を担当するエージェントです。

## タスク
J-Quants API Standardプラン用のPythonクライアントを実装してください。

## 要件
1. 認証処理（リフレッシュトークン/IDトークンの取得・更新）
2. 全エンドポイントへのアクセスメソッド
3. レート制限対応（1秒間隔）
4. 自動リトライ（exponential backoff）
5. 適切なエラーハンドリング
6. 型ヒント完備

## 使用ライブラリ
- requests
- python-dotenv
- tenacity（リトライ用）

## エンドポイント
- /token/auth_user, /token/auth_refresh（認証）
- /listed/info（銘柄情報）
- /prices/daily_quotes（株価）
- /fins/statements, /fins/announcement（財務）
- /indices, /indices/topix（指数）
- /markets/trades_spec（投資部門別）
- /markets/short_selling, /markets/margin_interest（信用取引）

## 注意
- uvを使用（pipは使わない）
- 環境変数から認証情報を取得
```

---

## Agent 2: Data Processing Agent

### 役割
データ取得・加工・キャッシュ処理を担当

### 担当ファイル
- `src/jquants_report/data/fetcher.py`
- `src/jquants_report/data/processor.py`
- `src/jquants_report/data/cache.py`
- `src/jquants_report/config.py`

### プロンプト
```
あなたはデータ処理モジュールの実装を担当するエージェントです。

## タスク
J-Quants APIから取得したデータの処理・キャッシュシステムを実装してください。

## 要件
1. APIクライアントを使用したデータ取得
2. pandas DataFrameへの変換
3. データの正規化・クレンジング
4. ローカルキャッシュ（Parquet形式）
5. キャッシュの有効期限管理
6. 差分更新の実装

## 取得するデータ
- 日次株価（全銘柄）
- 銘柄マスタ
- 指数データ
- 投資部門別売買状況
- 信用取引残高
- 決算発表予定

## 注意
- メモリ効率を考慮
- 型ヒント完備
- 適切なログ出力
```

---

## Agent 3: Analysis Agent

### 役割
市場分析ロジックの実装を担当

### 担当ファイル
- `src/jquants_report/analysis/market.py`
- `src/jquants_report/analysis/sector.py`
- `src/jquants_report/analysis/stocks.py`
- `src/jquants_report/analysis/technical.py`
- `src/jquants_report/analysis/supply_demand.py`
- `tests/test_analysis.py`

### プロンプト
```
あなたは市場分析モジュールの実装を担当するエージェントです。

## タスク
株式市場の各種分析ロジックを実装してください。

## 分析項目

### 市場概況 (market.py)
- 主要指数の騰落率計算
- 騰落銘柄数のカウント
- 市場全体の出来高・売買代金集計

### セクター分析 (sector.py)
- 33業種別騰落率計算
- 業種別ランキング生成

### 個別銘柄分析 (stocks.py)
- 値上がり/値下がり率TOP10
- 出来高急増銘柄検出（20日平均比2倍以上）
- ストップ高/安銘柄抽出
- 年初来高値/安値更新銘柄

### テクニカル指標 (technical.py)
- 騰落レシオ（25日）
- 移動平均乖離率
- 新高値/新安値銘柄数

### 需給分析 (supply_demand.py)
- 投資部門別売買動向サマリー
- 信用取引残高分析
- 貸借倍率計算

## 注意
- pandas/numpyを活用
- 計算結果はdataclassで返す
- ユニットテスト必須
```

---

## Agent 4: Report Generator Agent

### 役割
Markdownレポート生成を担当

### 担当ファイル
- `src/jquants_report/report/generator.py`
- `src/jquants_report/report/templates.py`
- `src/jquants_report/report/formatter.py`
- `tests/test_report.py`

### プロンプト
```
あなたはレポート生成モジュールの実装を担当するエージェントです。

## タスク
分析結果をMarkdown形式のレポートに変換する機能を実装してください。

## 要件
1. Jinja2テンプレートによるレポート生成
2. 各分析セクションのフォーマッター
3. 表形式データのMarkdownテーブル変換
4. 日付ベースのファイル出力

## レポート構成
1. 市場全体の概況
2. セクター分析
3. 個別銘柄ハイライト
4. 需給分析
5. テクニカル指標サマリー
6. 翌営業日の注目点

## 出力仕様
- ファイル名: market_report_YYYYMMDD.md
- 出力先: ./reports/
- 文字コード: UTF-8

## 注意
- 見やすいMarkdown形式
- 数値は適切にフォーマット（カンマ区切り、小数点以下桁数）
- 色分けやアイコンは使わず、テキストベースで表現
```

---

## 並列実行コマンド

```bash
# 4つのサブエージェントを並列起動
claude --subagent "API Client Agent" &
claude --subagent "Data Processing Agent" &
claude --subagent "Analysis Agent" &
claude --subagent "Report Generator Agent" &
wait
```

## 依存関係

```
Agent 1 (API) → Agent 2 (Data) → Agent 3 (Analysis) → Agent 4 (Report)
```

Agent 1とAgent 4は独立して開発可能。
Agent 2はAgent 1の完了後、Agent 3はAgent 2の完了後に統合テストを実施。

---
> Source: [ThreeBigTemple/jquants-daily-report](https://github.com/ThreeBigTemple/jquants-daily-report) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
