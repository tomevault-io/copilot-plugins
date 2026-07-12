## oyagami-local

> AI アシスタント向けプロジェクトガイドです。

# CLAUDE.md — oyagami-local（親神ローカル）

AI アシスタント向けプロジェクトガイドです。

## プロジェクト概要

知的障害・発達障害のある方の支援情報（ケアの暗黙知・禁忌事項・緊急連絡先など）を Neo4j グラフDBで管理し、ローカルLLM（Ollama）で分析・抽出を行うシステム。クラウドAPIを一切使用せず、個人情報をオンプレミスで完全管理する。隣接プロジェクト `../neo4j-agno-agent/` の機能をローカルLLM版として再実装したもの。

## アーキテクチャ

```
oyagami-local/               # モノリポ
├── backend/                 # FastAPI + Agno + Ollama
└── frontend/                # Next.js 15 + shadcn/ui + Tailwind CSS v4
```

- **バックエンド**: `backend/app/main.py` がエントリポイント。FastAPI + Agno フレームワーク。
- **フロントエンド**: `frontend/src/app/` に Next.js App Router のページが並ぶ。
- **データベース**: Neo4j 5.15。`../neo4j-agno-agent/docker-compose.yml` で起動する既存インスタンスを共用する。
- **LLM**: Ollama（`http://localhost:11434`）。モデルはタスク種別に応じてエージェントが使い分ける。

## 起動方法

```bash
# Neo4j（隣接プロジェクトの docker-compose を使用）
cd ~/Dev-Work/neo4j-agno-agent && docker-compose up -d

# バックエンド（ポート 8000）
cd ~/Dev-Work/oyagami-local/backend
uv run uvicorn app.main:app --reload --port 8000

# フロントエンド（ポート 3000）
cd ~/Dev-Work/oyagami-local/frontend
pnpm dev

# ワンコマンド起動
./scripts/setup.sh          # 全サービス起動
./scripts/setup.sh --stop   # 全停止
./scripts/setup.sh --status # 状態確認
```

## テスト

```bash
# バックエンドテスト
cd backend
uv run pytest tests/ -v

# E2Eテスト（Playwright）
cd frontend
npx playwright test
```

- パッケージ管理: **uv**（pip は使わない）
- フロントエンドのパッケージ管理: **pnpm**

## Phase 2 追加機能

- **セマンティック検索** — `nomic-embed-text` によるベクトル類似度検索。支援記録・ケア指示をキーワードではなく意味で検索できる。
- **エコマップ** — React Flowを使ったNeo4j Browser風のダーク背景グラフ表示。支援ネットワークを可視化する。
- **面談記録** — OpenAI Whisperによるローカル音声文字起こし。音声ファイルをアップロードするとテキスト化・embedding付与・Neo4j登録が一括実行される。ffmpeg が必要。

## 主要ディレクトリ

| パス | 内容 |
|---|---|
| `backend/app/agents/` | Agno マルチエージェント（team.py がオーケストレーター） |
| `backend/app/routers/` | FastAPI ルーター（chat, clients, dashboard, narratives, quicklog, search, system, ecomap, meetings） |
| `backend/app/lib/` | 共有ライブラリ（Neo4j 操作・embedding・chunking・transcription・ecomap） |
| `backend/app/schemas/` | Pydantic スキーマ定義 |
| `backend/app/config.py` | 設定管理（pydantic-settings、.env から読み込み） |
| `backend/scripts/backfill_embeddings.py` | 既存ノードへのembedding一括付与 |
| `backend/tests/` | pytest テスト |
| `frontend/src/app/` | Next.js App Router のページ |
| `frontend/src/components/` | shadcn/ui ベースのコンポーネント |
| `frontend/src/hooks/` | カスタムフック |
| `frontend/e2e/` | Playwright E2Eテスト |

## 主要 API エンドポイント（Phase 2 追加分）

| メソッド | パス | 説明 |
|---|---|---|
| POST | `/api/search/semantic` | セマンティック検索（ベクトル類似度） |
| GET | `/api/ecomap/templates` | エコマップテンプレート一覧 |
| GET | `/api/ecomap/colors` | カテゴリカラーマッピング |
| GET | `/api/ecomap/{name}` | エコマップデータ生成 |
| POST | `/api/meetings/upload` | 音声ファイルアップロード・文字起こし |
| GET | `/api/meetings/{name}` | 面談記録一覧 |

## バックフィルコマンド

```bash
# 既存ノードへの embedding 一括付与
cd backend
uv run python scripts/backfill_embeddings.py
```

## マルチエージェント構成

| エージェント | ファイル | モデル | 役割 |
|---|---|---|---|
| Coordinator | `agents/coordinator.py` | mistral-small | 意図分類・ルーティング（常駐） |
| Intake | `agents/intake.py` | deepseek-r1:70b | テキスト→JSON抽出（排他） |
| Validator | `agents/validator.py` | mistral-small | 論理検証・安全性チェック（常駐） |
| Analyst | `agents/analyst.py` | llama4 | 分析・支援方針策定（排他） |
| CypherGen | `agents/cypher_gen.py` | qwen3-coder:30b | Cypher クエリ生成（排他） |
| Team | `agents/team.py` | — | AgnoTeam オーケストレーター |

排他モデルは同時に1つしかロードされない。ModelManager がメモリ管理を担う。

## Neo4j スキーマ規約

`docs/SCHEMA_CONVENTION.md`（`~/Dev-Work/shared-schema/` から同期される正典コピー）に準拠する。

- **ノード**: PascalCase（例: `Client`, `NgAction`, `SupportLog`）
- **リレーション**: UPPER_SNAKE_CASE（例: `MUST_AVOID`, `HAS_KEY_PERSON`, `LOGGED`）
- **プロパティ**: camelCase（例: `riskLevel`, `nextRenewalDate`, `createdAt`）
- クエリは必ずパラメータ化（`$param`）してインジェクションを防ぐ
- 冪等性のために `MERGE` を使用する

主要ノード: `Client`, `NgAction`, `CarePreference`, `Condition`, `KeyPerson`, `Guardian`, `Hospital`, `Supporter`, `SupportLog`, `MeetingRecord`, `Certificate`

## 重要な制約

### Safety First
- 緊急時の情報取得は **LLM を経由しない**。Neo4j を直接検索して NgAction を返す。
- `NgAction` ノードの `riskLevel` は `LifeThreatening` > `Panic` > `Discomfort` の優先順位。

### データ完全性
- AI 抽出はテキストに存在しない情報を補完・推測してはならない（**No Fabrication**）。
- クライアント名は一意制約あり。登録前に `validate_client_uniqueness()` で確認する。

### モデル選択
- 環境変数（`.env`）でモデルを設定する。ハードコード禁止。
- モデル名は `config.py` の設定クラス経由で参照する。

### フロントエンド
- 日本語名の検索UIはテキスト入力を避け、あかさたな方式のボタンフィルタを使う（`st.text_input` 相当の IME 問題を回避するため）。

### 音声文字起こし（Whisper）
- Whisper による音声文字起こしには **ffmpeg** が必要（`brew install ffmpeg`）。
- ffmpeg がインストールされていない場合、面談記録の音声アップロード機能は動作しない。

---
> Source: [kazumasakawahara/oyagami-local](https://github.com/kazumasakawahara/oyagami-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
