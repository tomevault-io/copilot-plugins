## koe

> 音声をテキストに文字起こしし、トピックごとに分割するデスクトップアプリ + API + MCP サーバー。

# koe

音声をテキストに文字起こしし、トピックごとに分割するデスクトップアプリ + API + MCP サーバー。

## Architecture

### 全体構成

```
Desktop App (Electron + React)
    ├─ 録音 (MediaRecorder)
    ├─ Web Audio API でデコード → 16kHz mono にリサンプリング → 60s チャンクに分割 (WAV)
    ├─ ジョブ作成 (multipart で chunk 群 + メタ JSON)
    └─ ジョブポーリング
         ▼
Workers (Hono/TS) ── API / 認証 (すべて requireAuth)
    ├── R2 (chunk 音声 + 結果 JSON)
    ├── D1 (ジョブ/チャンク/トピックのメタ情報)
    └── DurableObject (KoeProcessor) ── alarm パターンで非同期ジョブ処理
            ├── Whisper (Workers AI、chunk 単位で順次直呼び)
            └── トピック分割 (Gemini Flash)
```

### パッケージ構成（モノレポ）

```
packages/
├── api/       # Workers + Hono (TS) - API / 認証 / MCP / Whisper・Gemini オーケストレーション
├── shared/    # 共有ユーティリティ (format / auth / API client)
└── desktop/   # Electron デスクトップアプリ (録音 + チャンク分割)
```

### 技術スタック

| レイヤー     | 技術                                                                          |
| ------------ | ----------------------------------------------------------------------------- |
| API          | Cloudflare Workers + Hono (TypeScript)                                        |
| 音声分割     | Desktop の Web Audio API (16kHz mono / WAV)                                   |
| 文字起こし   | Workers AI Whisper (@cf/openai/whisper-large-v3-turbo)、Workers TS から直呼び |
| トピック分割 | Gemini Flash（デフォルト）、Workers TS から直呼び                             |
| DB           | Cloudflare D1                                                                 |
| Storage      | Cloudflare R2                                                                 |
| 認証         | Google OAuth (Device Flow) → JWT                                              |
| フロント     | Electron + React (デスクトップアプリ)                                         |

### API 設計

すべて requireAuth。Desktop も MCP もログイン必須。

```
# ジョブ
POST   /api/v1/jobs              # ジョブ作成（multipart で chunks_meta JSON + chunk_N WAV 群）→ DO で非同期処理
GET    /api/v1/jobs              # ジョブ一覧（自分のだけ）
GET    /api/v1/jobs/:id          # ジョブ詳細
DELETE /api/v1/jobs/:id          # ジョブ削除（R2 → D1 の順）
POST   /api/v1/jobs/:id/analyze  # analyze だけ再実行（transcribed / analyze_failed のみ）
GET    /api/v1/jobs/:id/topics   # トピック一覧

# R2 Multipart Upload（大容量音声、将来用）
POST /api/v1/uploads                        # アップロード開始
PUT  /api/v1/uploads/:uploadId/parts/:num   # パーツアップロード
POST /api/v1/uploads/:uploadId/complete     # 完了
DELETE /api/v1/uploads/:uploadId            # 中止

# 認証
GET  /auth/device              # Device Flow 開始
POST /auth/token               # トークン交換

# MCP（Streamable HTTP、認証必須）
ALL  /mcp                      # Codex Desktop / モバイル向けリモート MCP
```

### MCP ツール

`/mcp` は `requireAuth` 下で以下 4 ツールを公開（参照系のみ、ユーザーごとにデータ分離）:

| Tool            | 説明                                       |
| --------------- | ------------------------------------------ |
| `list_jobs`     | 自ユーザーのジョブ一覧                     |
| `get_job`       | ジョブ詳細（他ユーザーのジョブは返さない） |
| `get_topics`    | ジョブに紐づくトピック一覧                 |
| `search_topics` | タイトル/サマリ LIKE 検索、自ユーザー範囲  |

### R2 キー設計

```
{userId}/audio/{jobId}/chunks/{index}.wav     # Desktop で事前分割した chunk
{userId}/results/{jobId}/transcript.json
{userId}/results/{jobId}/topics.json
```

### ジョブ状態遷移

```
pending → transcribing → transcribed → analyzing → completed
                  ↓                ↓
        transcribe_failed   analyze_failed
                                  ↑              ↓
                                  └─ POST /jobs/:id/analyze で再実行可能
```

- 各 phase 独立で max 3 回までリトライ、線形 backoff (30/60/90s)
- analyze 失敗時は transcript を R2 に保持したまま、Whisper を再課金せず Gemini だけ再実行できる

### 設計方針

- Whisper / LLM クライアントは Workers AI / Gemini REST を直接呼び、baseURL 差し替えで AI Gateway に通せる
- D1 にはメタ情報のみ、巨大テキストは R2 に JSON で保存
- 長時間音声は Desktop 側で 60s 単位の時間分割（静音検出は使わない / Whisper + Gemini で実用上問題なし）
- 大容量ファイルは R2 Multipart Upload で対応（Desktop v1 では未使用）
- ジョブは冪等性を担保（チャンクID + 状態管理）
- transcribe (Whisper) と analyze (Gemini) は独立 phase で動く。transcribe 完了時点で transcript.json を R2 に commit し、analyze で失敗しても Whisper を再課金しない
- 進捗通知はポーリング + ステータスAPI
- BYOK 対応: Desktop の SettingsPanel で Gemini / CF キーを入力すると、リクエストヘッダ (`X-Gemini-Key` 等、`@koe/shared/byok-headers` で定義) 経由で env 既定値を上書きする。未入力なら env 既定値で動く。キーは Desktop 側では Electron `safeStorage` に暗号化保存し、サーバ側ではジョブ処理中のみ DurableObject storage に保持する（CF の at-rest 暗号化下）。ジョブ完了 / 全リトライ失敗時に `ctx.storage.delete("job")` で破棄され、長期保管はしない

## Coding Rules

- パッケージマネージャーは pnpm を使用
- コード修正後は test / lint / format を実行
- テスト駆動開発（TDD）で進める（Red → Green → Refactor）
- YAGNI 原則に従い余計な機能は実装しない

---
> Source: [owk-owk130/koe](https://github.com/owk-owk130/koe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
