## lcl-ai-pg-vector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 使用說明

當使用者詢問「這個環境怎麼用？」、「有哪些功能？」、「可以做什麼？」等問題時，請讀取 `GUIDE.md` 並以友善的方式呈現其內容。

## Project Overview

VLA PlayGround（Visual Landscape Analysis PlayGround）是一個「進入即可用」的 AI 研究環境。研究者啟動 `claude` 後，所有 AI 服務自動就緒。

核心服務 **LaDeco** 是基於 OneFormer 模型（ADE20K 資料集）的景觀影像語意分割工具，將 ADE20K 150 類映射為自訂 4 層階層標籤系統，計算面積比例與自然特徵指數（LC_NFI）。

Reference: Li-Chih Ho (2023), *LaDeco: A Tool to Analyze Visual Landscape Elements*, Ecological Informatics, vol. 78.

## 可用服務

### LaDeco — 景觀影像語意分割

MCP 工具（9 個）：`predict`, `predict_batch`, `predict_images`, `visualize`, `get_area`, `list_datasets`, `preview_dataset`, `get_color_legend`, `get_label_hierarchy`

- `predict`, `predict_images`, `get_area` 預設 `level=2`（L2）
- 呼叫前應詢問使用者想用哪一層（L1–L4），若未指定則用預設 L2
- REST API: `http://192.168.1.162:30800/api/` (Swagger: `/api/docs`)

### NAS File Server — 檔案管理

MCP 工具（8 個）：`list_files`, `read_file`, `write_file`, `delete_file`, `move_file`, `copy_file`, `file_info`, `search_files`

- REST API: `http://192.168.1.162:30803/api/` (Swagger: `/api/docs`)
- 多檔上傳（不經 LLM）: `POST /api/upload/{directory}`

### pg-vector — 研究資料庫 + 向量搜尋

PostgreSQL + pgvector 資料庫，管理研究專案、影像中繼資料、分析結果，並支援向量相似度搜尋（以圖搜圖）。部署於 NAS02 (192.168.1.152)。

MCP 工具（13 個）：`create_project`, `get_project`, `list_projects`, `delete_project`, `create_image`, `get_image`, `list_images`, `update_image_embedding`, `create_analysis_result`, `list_analysis_results`, `search_similar_images`, `search_similar_by_image_id`, `get_project_summary`

- REST API: `http://192.168.1.152:30805/api/` (Swagger: `/api/docs`)
- Schema: projects, images (with vector(512) embedding), analysis_results
- 向量搜尋使用 HNSW 索引 + cosine similarity

## 標籤階層（L1–L4）

| 層級 | 說明 | 類別數 | 範例 |
|------|------|--------|------|
| L1 | 自然 vs. 人造 | 2 | nature, man_made |
| L2 | 主要類別 | 7 | bio, landform, sky, vegetation, water, archi, street |
| L3 | 中等細分 | 16 | herb_plant, woody_plant, hori_water, vert_water… |
| L4 | 精細類別 | ~50 | tree, grass, road, building, car, river… |

L4 labels map to ADE20K indices, then L3/L2/L1 aggregate by tuple concatenation (`_get_ladeco_labels()` in `engine.py`).

## Git Workflow

- 本專案有兩個 remote：`origin`（GitHub）和 `gitlab`（GitLab）
- Commit 後 push 必須同時推送到兩個 remote：`git push origin main && git push gitlab main`

## Key Conventions

- NAS datasets mount at `/mnt/ai_data` (read-only in compose.yml)
- Temporary CSV results go to `/tmp/LaDeco-*.csv`, cleaned weekly by cron inside the container
- The Docker healthcheck pings `/healthcheck` every 15 minutes
- `.env` configures `HOST_IP`, `HOST_PORT`, `DATABASES_ROOT`, `COMPOSE_PROJECT_NAME`
- The project has no test suite or linter configured
- `core.py` is a backward-compatibility shim (`from engine import *`)

## 詳細文件索引

| 文件 | 內容 |
|------|------|
| [`docs/services/ladeco.md`](docs/services/ladeco.md) | LaDeco 架構、Build & Run、REST API、部署 |
| [`docs/services/nas-files.md`](docs/services/nas-files.md) | NAS File Server 架構、REST API、部署 |
| [`docs/services/pg-vector.md`](docs/services/pg-vector.md) | pg-vector 架構、Schema、REST API、部署 |
| [`docs/infrastructure.md`](docs/infrastructure.md) | 基礎設施（IP、K8s、GitLab CI/CD、Registry） |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lichihho)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lichihho)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
