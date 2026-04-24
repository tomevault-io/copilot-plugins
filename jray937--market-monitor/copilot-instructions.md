## market-monitor

> - **名稱**: market-monitor

# CLAUDE.md

## 專案概述
- **名稱**: market-monitor
- **類型**: Multi-Agent Discord Bot（Python 3.11 + discord.py）
- **功能**: 7 Bot 投研團隊（1 Leader + 6 Agent），LLM 驅動調度，美股/加密監控
- **GitHub**: github.com/Jray937/market-monitor

---

## 架構

- ** Railway 單一服務**，7 個 Bot 以 Python `threading` 同時運行
- **Leader Bot** = LLM 驅動的智能調度員（MiniMax），接收需求 → LLM 分析意圖 → 調度 Agent → 彙總回覆
- **Agent Bot** × 6：trader / sector_analyst / macro_strategist / intelligence_officer / risk_officer / quant_strategist
- 協調方式：共享「團隊頻道」，透過訊息格式協作（`[Agent名稱] 任務ID 報告內容`）

---

## 技術棧
- Python 3.11
- discord.py（Bot，支援 slash commands + on_message）
- yfinance（市場數據）
- anthropic（MiniMax API，Anthropic SDK 相容）
- APScheduler（排程）

---

## 環境變數

**必要：**
| 變數 | 說明 |
|------|------|
| `LEADER_BOT_TOKEN` | Leader Bot Discord Token |
| `TEAM_CHANNEL_ID` | 團隊協調頻道 ID |
| `MINIMAX_API_KEY` | MiniMax API Key |
| `MINIMAX_API_BASE_URL` | `https://api.minimaxi.com/anthropic` |
| `MINIMAX_MODEL` | `MiniMax-M2.7` |

**各 Agent Token（否則該 Bot 不啟動）：**
`CHIEF_STRATEGIST_TOKEN` / `TRADER_TOKEN` / `SECTOR_ANALYST_TOKEN` / `MACRO_STRATEGIST_TOKEN` / `INTELLIGENCE_OFFICER_TOKEN` / `RISK_OFFICER_TOKEN` / `QUANT_STRATEGIST_TOKEN`

---

## 🚨 開發流程鐵律

1. **開分支** → `git checkout -b fix/xxx`
2. **本地測試** → `python3 -m py_compile bot.py src/*.py`
3. **提交 PR** → `gh pr create`
4. **Merge** → `gh pr merge --squash --delete-branch`

---

## 核心模組

- `bot.py` — 入口，呼叫 `discord_bot.main()`
- `src/discord_bot.py` — **全部 Bot 邏輯（Leader + 所有 Agent）**
  - `run_leader_bot()`：LLM 調度 + 狀態追蹤
  - `run_team_agent_bot()`：監聽團隊頻道 + AI 分析
  - `leader_analyze()`：LLM 意圖分析（action=answer/dispatch/hybrid）
  - `TaskState`：狀態追蹤類別
- `src/config.py` — 讀取 config.yaml + `load_agents_config()`

---

## 狀態流程

```
📤 發送中 → ✅ 已接收 → 🔄 處理中 → 📝 匯總中 → ✅ 完成
                    ↓（120 秒無回應）
                 ❌ 接收超時
```

---

## Leader LLM 調度

每次用戶輸入都調用 `leader_analyze()` → MiniMax LLM → JSON 決策：
```python
{
  "action": "dispatch|answer|hybrid",
  "agents": ["trader", "risk_officer"],
  "task": "任務描述",
  "direct_answer": "結論（answer/hybrid時）"
}
```

---

## 已知問題 / 陷阱

- **@mention 不可靠**：用 `client.user.id in {int(m.id) for m in message.mentions}` 而非 `client.user in message.mentions`
- **Interaction 超時**：slash command 需用 `defer(thinking=True)` 再 `followup.send()`
- **Agent 全部要加入同一伺服器** 並有團隊頻道讀/發權限
- **Message Content Intent**：每個 Bot 的 Application 都必須開啟此權限

---

## 2026-03-25 變更記錄

### 架構重構
- `config.yaml` `agents:` 區塊恢復（6 個 Agent 終於能啟動）
- Leader Bot `on_message` 區分團隊頻道（接收報告）vs 用戶頻道（接收需求）
- `pending_tasks` 終於被填充（之前 reports 永遠是空）

### LLM 驅動調度
- 廢除正則匹配股票代碼的舊邏輯
- 新增 `leader_analyze()` 調用 MiniMax LLM 解析任意用戶輸入
- 三種模式：answer（直接回答）/ dispatch（調度團隊）/ hybrid（先結論後團隊）
- 動態調度：根據問題判斷調度哪些 Agent，非全部

### 狀態追蹤
- `TaskState` 類別：每個 Agent 獨立狀態追蹤
- Agent 收到任務後回傳 `✅ 已接收任務` 確認
- `monitor_task()` 協程處理超時（120 秒接收超時）

### `/ask` 命令
- 新增主力 slash command `/ask <問題>`
- 使用 `defer(thinking=True)` 避免 interaction 過期
- 與 `leader_analyze()` 共用同一套 LLM 調度邏輯

### Debug
- `on_message` 加入日誌：`📨 on_message: author=..., is_dm=..., is_mentioned=...`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jray937) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
