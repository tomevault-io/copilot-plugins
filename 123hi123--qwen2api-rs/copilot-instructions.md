## qwen2api-rs

> 把 Qwen Web 偽裝成 OpenAI/Anthropic/Gemini 相容介面的網關。**後端 Rust（axum/tokio/reqwest）+ 純原生前端三檔（web/）**。

# qwen2api-rs — 專案 AI 上下文

把 Qwen Web 偽裝成 OpenAI/Anthropic/Gemini 相容介面的網關。**後端 Rust（axum/tokio/reqwest）+ 純原生前端三檔（web/）**。
基於上游 `YuJunZhiXue/qwen2API` 重寫；基準版本與同步流程見 `dev/UPSTREAM.md`、協議見 `dev/PROTOCOL.md`、架構見 `dev/ARCHITECTURE.md`。
所有可調 env 變數的權威清單在 `.env.example`（含風控/帳號池/上下文等 20+ 項）。

## 開發 / 執行注意
- `CARGO_TARGET_DIR=/home/joe/.cache/cargo-target` → 二進位在該處的 `debug/`、`release/`，**不是** `./target`。
- 前景 `sleep` 被 harness 阻擋（exit 144）→ 啟動長駐服務用背景任務 + `curl --retry --retry-connrefused` 等就緒。
- 本機測試用 port 7866、`ADMIN_KEY=testadmin123`、`data/` 放少量真實帳號；正式部署在 7860（見下）。
- 真實測試帳號來源：`/home/joe/文件/docker/qwen2API/data/accounts.json`（約 16,857 個）。
- 本機開發**不要**用根目錄 `docker-compose.yml`（那份是部署範例、用 7860）；直接 `cargo run` 讀 `.env`（已預設 PORT=7866、ADMIN_KEY=testadmin123）。正式部署看下方。

## 部署（Docker，正式環境）
**部署位置：`/home/joe/文件/docker/qwen2api-rs/`**（與原 Python 版 `qwen2API` 並列）。
- 該資料夾只放 `docker-compose.yml` + `data/`（accounts.json / api_keys.json / users.json）；**源碼留在本 dev 專案**，compose 以 `build.context: ../../dev/qwen2api-rs` 從這裡建置。
- 對外端口 **7860**（取代原 Python 版）。`ADMIN_KEY` 沿用原版值，使既有 API Key 不失效。
- ⚠️ **風控代理必帶**：原版 `docker-compose.override.yml` 讓出口走 `HTTP(S)_PROXY=http://ramdon:joe@192.168.1.203:2260`（隨機出口 IP）。新 compose 的 `environment` 必須設相同 `HTTP_PROXY/HTTPS_PROXY/NO_PROXY`；reqwest 預設會讀這些 env，無需改碼。
- 完整遷移/部署 runbook 見 **`dev/DEPLOY.md`**。

## 🔁 [流程] 每次開發完成後的重新部署
開發改完、`cargo build` 通過、本機(7866)驗證 OK 後：
```bash
cd /home/joe/文件/docker/qwen2api-rs
docker compose up -d --build      # 從 ../../dev/qwen2api-rs 重新建置並滾動更新
docker compose logs -f --tail=30  # 確認啟動 + 帳號載入
curl -s http://127.0.0.1:7860/healthz   # 應 200
```
資料（帳號/金鑰）在 `data/` 持久化，重部署不丟。完成後依全域規則對「服務部署完成」發 tg-notify。

---
> Source: [123hi123/qwen2api-rs](https://github.com/123hi123/qwen2api-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
