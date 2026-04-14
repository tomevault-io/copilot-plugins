## secretary-bot

> - User writes in Japanese; **respond in Japanese**.

# CLAUDE.md

## Communication
- User writes in Japanese; **respond in Japanese**.
- Write lightweight, efficient code. Prefer minimal dependencies.

## Work Location Detection
- Working in `D:/Git/` → **Home (Sub PC)**
- Working in `C:/Git/` → **Home (Main PC)**
- Working in `C:/Users/**/Documents/git/` → **Remote PC**
  - Remote PC lacks required environments. Focus on code adjustments only.
- Can SSH into Raspberry Pi via `ssh iniwapi` to read code/logs from the Pi

## Environments

### Raspberry Pi 4 (Bot Host)
- RAM: 8GB, Arch: `linux/arm64`
- Docker management: Portainer — Stack Web Editor only
- SSH: `ssh iniwapi`

### Main PC
| Item | Detail |
|------|--------|
| CPU | Ryzen 7 9800X3D |
| GPU | RTX 4080 — CUDA available |
| RAM | 48GB |
| OS | Windows 11 |
| IP | 192.168.1.210 |

### Sub PC
| Item | Detail |
|------|--------|
| CPU | Ryzen 9 5950X |
| GPU | RTX 5060 Ti — 16GB VRAM, CUDA Compute 8.9 (sm_89) |
| RAM | 64GB |
| OS | Windows 11 |
| IP | 192.168.1.211 |

### Windows PC共通
- Ollama + Windows Agent（Dockerなし・Pythonネイティブ）
- Ollama: `:11434`, Windows Agent: `:7777`

## Build & Deploy
- Build target: `linux/arm64`
- Image: `ghcr.io/iniwa/secretary-bot:latest`
- **イメージにはPythonランタイムとライブラリのみ**を含める（コードは含めない）
- GitHub Actionsのトリガーは `Dockerfile` と `requirements.txt` の変更時のみ
- コード変更時はWebGUIの「コード更新」ボタン（git pull + Portainer API再起動）で対応
- Dockerfileに `git` のインストールが必要（WebGUIからgit pullを実行するため）
- Resource limits: `deploy.resources.limits.memory` を検討（8GB共有）

## Storage
| Data | Path | Backend |
|------|------|---------|
| ソースコード（Volume） | `/home/iniwa/docker/secretary-bot/src` | SSD |
| 設定ファイル（Volume） | `/home/iniwa/docker/secretary-bot/config.yaml` | SSD |
| SQLite / ChromaDB | `/home/iniwa/docker/secretary-bot/data` | SSD |

## 環境変数
- APIキー・トークン等の機密情報はすべて `.env` で管理
- `config.yaml` には機密情報を含めない
- `.env.example` にダミー値のテンプレートを用意

## Architecture

```
[Discord] [WebGUI]
    ↓         ↓
[Skill Router]        ← LLMがどのユニットを使うか判断（JSON返却）
    ↓
[Unit Manager]        ← ユニットを自動ロード・管理
    ├── Pi上のUnit    → そのまま実行
    └── DELEGATE_TO="windows" → RemoteUnitProxy → AgentPool → Windows Agent

[Heartbeat]           ← 適応型頻度制御（Ollama有無で間隔切り替え）
[LLM Router]          ← Ollama優先 → Gemini APIフォールバック
[AgentPool]           ← 複数Windows PCをpriority順に管理
[SQLite]              ← 全データ永続化
[ChromaDB]            ← インプロセス（PersistentClient）・ベクトル記憶
[WebGUI]              ← FastAPI + レスポンシブHTML（PCファースト）
```

## Project Structure

```
secretary-bot/
├── src/
│   ├── bot.py                # エントリーポイント
│   ├── skill_router.py       # 自然言語 → ユニット振り分け
│   ├── heartbeat.py          # ハートビート・コンテキスト圧縮
│   ├── errors.py             # BotError基底クラス
│   ├── circuit_breaker.py    # サーキットブレーカー
│   ├── logger.py             # 構造化ログ（JSON・trace_id）
│   ├── database.py           # SQLite（aiosqlite・WAL）
│   ├── llm/
│   │   ├── router.py         # Ollama/Gemini切り替え
│   │   ├── ollama_client.py
│   │   └── gemini_client.py
│   ├── memory/
│   │   ├── chroma_client.py  # ChromaDB（PersistentClient）
│   │   ├── ai_memory.py      # AI自身の記憶（Ollama専用）
│   │   └── people_memory.py  # 人物記憶（Geminiフォールバック可）
│   ├── units/
│   │   ├── base_unit.py      # BaseUnit（Cog継承）
│   │   ├── remote_proxy.py   # 透過的な委託ラッパー
│   │   ├── agent_pool.py     # 複数PC管理
│   │   ├── reminder.py / memo.py / timer.py / status.py / chat.py
│   └── web/
│       ├── app.py            # FastAPI（WebGUI + /health）
│       └── static/
├── windows-agent/
│   ├── agent.py              # FastAPI（:7777）
│   ├── units/
│   └── start_agent.bat
├── Dockerfile
├── config.yaml.example
├── .env.example
├── docker-compose.yml
└── .claudeignore
```

## Key Rules

### Unit追加
1. `src/units/my_unit.py` を作成し `BaseUnit` を継承
2. `config.yaml` の `units:` に追記 → 自動ロード

### Discord通知
- 各ユニットが `BaseUnit` のヘルパー（`notify()` / `notify_error()`）経由で直接送信
- 「Discordユニット」は作らない

### Windows委託
- `DELEGATE_TO = "windows"` の1行で委託化
- バージョンチェック方式（`GET /version` → 不一致時のみ `POST /update`）

### LLM利用方針
- `ai_memory` 書き込みはOllama必須（Gemini不可）
- Geminiトグルは全項目デフォルトOFF（高額課金防止）
- 省エネモード時: ペルソナ注入なし、`people_memory` のみ注入

### エラーハンドリング
- 全エラーは `BotError` 継承
- サーキットブレーカーで保護（連続失敗 → 一時停止 → 自動復帰）
- 構造化ログに `trace_id` 付与

### キャラクター（ミミ）
- 一人称「僕」固定、フレンドリーな砕けた敬語
- 理論的・現実主義・ちょっとだけ毒舌
- 禁止: 過度な甘やかし・無駄な称賛・感情的な発言
- 詳細は `config.yaml` の `character.persona` に定義

## Security

GitHubパブリックリポジトリのため:
- `.env` / `config.yaml`（実体）/ `data/` / `*.key` / `*.pem` を `.gitignore` に含める
- APIキー・トークン・IPアドレスを**コードにハードコードしない**
- コンテナは非root（`botuser`）で実行
- Windows Agent認証: `X-Agent-Token` ヘッダー（トークンは `.env` 管理）
- WebGUI: Basic認証必須
- 外部公開時: Cloudflare Tunnel + Cloudflare Access

## Tech Stack
| 項目 | 採用 |
|------|------|
| 言語 | Python 3.11 |
| Discord | discord.py v2 |
| Web | FastAPI |
| DB（構造化） | SQLite（aiosqlite・WAL） |
| DB（記憶） | ChromaDB（PersistentClient） |
| スケジューラ | APScheduler |
| LLM | Ollama API（qwen3）/ Google Generative AI SDK |

## Tooling
- Use **Serena MCP** tools for code navigation and editing to maximize efficiency (symbol search, overview, replace, insert, etc.)

## Knowledge Persistence
- 設計判断・知見は `docs/*.md` に積極的に残す
- 作業開始時に `docs/` の既存コンテキストを確認する

## Checklist
- [ ] arm64-compatible base image (`python:3.11-slim`)
- [ ] Non-root user (`botuser`)
- [ ] `TZ=Asia/Tokyo` in environment
- [ ] `restart: unless-stopped`
- [ ] `healthcheck:` in docker-compose.yml
- [ ] `env_file:` in docker-compose.yml
- [ ] Image: `ghcr.io/iniwa/secretary-bot:latest`
- [ ] GitHub Actions at `.github/workflows/docker-publish.yml`
- [ ] `.env.example` / `config.yaml.example` in project root
- [ ] `.claudeignore` in project root
- [ ] Graceful shutdown（SIGTERM → DB close → Bot close）
- [ ] `/health` endpoint on FastAPI
- [ ] Verify deployment via Portainer Stack

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iniwa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iniwa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
