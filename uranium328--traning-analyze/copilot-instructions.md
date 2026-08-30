## traning-analyze

> 抓運動平台的騎乘活動 → 清洗 → 呼叫 LLM 生成中文教練報告與 4 天訓練計畫 →

# CLAUDE.md — AI 虛擬自行車教練

## 專案一句話

抓運動平台的騎乘活動 → 清洗 → 呼叫 LLM 生成中文教練報告與 4 天訓練計畫 →
推播 Discord。由 GitHub Actions 每小時執行（`.github/workflows/coach.yml`）。

**目前狀態（2026-07）**：已完成從 Strava 到 Garmin Connect 的資料源重構。
現以 `garminconnect`（0.3.6，需 Python ≥ 3.12）抓取騎乘活動與健康／恢復數據；
認證用本機 `login_setup.py` 匯出的 token（`GARMINTOKENS` secret，效期約一年，
不再需要 `GH_PAT` 回寫）。有最小測試套件：`python -m pytest tests/`。
原始重構計畫存於 [docs/garmin-refactor-plan.md](docs/garmin-refactor-plan.md)（歷史參考；
實作以程式碼為準——計畫書寫的 garth `dumps()` API 已過時，實作改用 `garmin.client.dumps()`）。

## 指令

- 執行主流程：`python main.py`（需要 `.env` 內的 `GARMINTOKENS`／`OPENAI_API_KEY`／
  `DISCORD_WEBHOOK_URL`；本機未設時 Garmin 登入或環境檢查會退出，屬正常）
- 安裝依賴：`pip install -r requirements.txt`
- 首次取得 Garmin token：本機跑一次 `python login_setup.py`。
- 測試：`python -m pytest tests/`（假資料 smoke test，不需帳號或網路）。改動核心邏輯時，
  除跑測試外，仍讓 verifier agent 實跑受影響模組（見 dispatch.md）。

## 紅線（違反 = 事故）

1. `tokens.json`、`.env`、API key、Discord webhook URL 的**內容**不得出現在
   commit、log 輸出或對話文字裡。
2. 不主動 commit/push；使用者要求才做。訊息含 `[skip ci]` 的 commit 是 CI
   機器人寫的（每小時更新 `processed_ids.txt`），不要 revert 或誤認為人為改動。
3. `processed_ids.txt` 是狀態檔，由程式與 CI 維護，不要手動編輯。

## Shell 速查（Windows 11 + PowerShell 5.1）

- POSIX 語法（`&&`、管線組合、heredoc）→ 用 Bash tool；PowerShell 5.1 沒有 `&&`。
- PowerShell 寫檔必加 `-Encoding utf8`（預設 UTF-16 會害 Python 讀不了）。
- 同一目的的指令失敗兩次 → 停止重試，改讀文件或換方法（兩振規則）。

## 規則檔路由（.claude/rules/）

| 時機 | 讀哪份 |
|---|---|
| 每個 session 開工前、要派 subagent 前 | [dispatch.md](.claude/rules/dispatch.md) — 模型調度：誰下場、派工格式、升降級、驗證 |
| 拿不定主意（要不要問使用者、算不算完成、要不要換方向、要不要升級模型） | [judgment.md](.claude/rules/judgment.md) — 判斷 rubric，附正反例 |
| 要派工時直接抄模板 | [templates.md](.claude/rules/templates.md) — 搜尋/實作/重構/研究/審查五種 |
| 要修改任何規則檔或 CLAUDE.md 之前 | [maintenance.md](.claude/rules/maintenance.md) — 什麼能自己改、備份規則、教訓格式 |
| 新 session 第一次接手這個環境 | [letter.md](.claude/rules/letter.md) — 前任留下的環境重點與制度退化警告 |
| 想了解這些規則為什麼存在 | [diagnosis.md](.claude/rules/diagnosis.md) — 三大失效模式診斷 |

規則衝突時的優先序：使用者當下指示 > 本檔紅線 > dispatch.md > judgment.md
> maintenance.md > templates.md。（letter.md 與 diagnosis.md 是背景說明，
不具規則效力。）仍無法裁決時，選動作較小、較可逆的那條，並在總結標註。

---
> Source: [uranium328/traning_analyze](https://github.com/uranium328/traning_analyze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
