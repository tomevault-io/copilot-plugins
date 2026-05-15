## alfred-butler

> **Service：** `alfred.service`

# Alfred 阿福 — 開發入口

**描述：** 零介面語音管家 App
**Service：** `alfred.service`
**Port：** `9001`（只動阿福，不碰其他服務）
**後端：** `/opt/alfred/backend/main.py`
**iOS 專案：** `Alfred.xcodeproj`（此 repo 根目錄）

## ⚠️ 開工前必讀

1. `CRITICAL_README.md` — 強制規則
2. `ALFRED_SOUL.md` — 阿福人格、語氣、零介面哲學
3. `HANDOFF.md` — 後端 API / VPS 架構
4. `SCENARIOS.md` — 家管、辦公室、寵物、運動等北極星情境

## 核心鐵律

### 1. 零介面
阿福不是聊天 App。平常只有語音，不顯示文字對話流。
只有主人必須「看」內容時才出現介面：文件、合約、圖片、翻譯大字、授權。

### 2. 管家邏輯
先理解、先生成草案，再問主人是否要下一步。不要把生活規劃一開始就變成授權流程。

### 3. 不重複問授權
Google 已授權時，先查連線狀態，不要反覆推 OAuth。

### 4. 不幻覺
只能引用工具實際查到的結果。找不到就說找不到。

### 5. 程式改動沙盒原則
所有改動在 git worktree 裡進行，測試確認後才 merge 回 main。

## 後端維護

```bash
systemctl restart alfred
systemctl status alfred --no-pager -l
curl -sS http://127.0.0.1:9001/health
```

不要 kill 其他 port/process。`8000` = 股市交易所，`9001` = 阿福。

## 場景模式
- 辦公室 GPS → `mode=work`，優先會議、文件、行程、待辦
- 家中 → `mode=home`，優先家人安全、寵物照顧
- 海外 → `mode=travel`，優先翻譯、交通、安全

---
> Source: [norika1207-lab/alfred-butler](https://github.com/norika1207-lab/alfred-butler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
