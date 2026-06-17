## save-your-token

> 使用極致省 Token 模式分析網頁內容。自動清洗廣告與雜訊，適合分析長篇文章。


### 概述
本技能提供極致省 Token 的網頁內容分析功能，透過外部 Python 腳本 (`eco_engine.py`) 與 `trafilatura` 進行內容清洗，並使用 `google-genai` SDK 進行摘要。

### 安裝需求
1. 確保 Python 虛擬環境 (`venv`) 已安裝必要的依賴：
   ```bash
   pip install trafilatura google-genai
   ```
2. 確保執行腳本具備正確的可執行權限。

### 設定步驟
1. 在專案中放置 `eco_engine.py` (包含 API Key 設定)。
2. 在 `.hermes/skills/eco_analyzer/manifest.json` 中設定執行指令：
   ```json
   {
     "handler": {
       "type": "shell",
       "command": "/path/to/venv/bin/python3 /path/to/eco_engine.py \"{{url}}\" \"{{query}}\""
     }
   }
   ```

### 最佳實踐
- **SDK 版本**：請使用最新的 `google.genai` 套件，取代過時的 `google.generativeai`。
- **內容處理**：若內容過長，請在傳送給模型前進行截斷（如範例中的 `[:5000]`）。
- **除錯**：若遇 `400 INVALID_ARGUMENT`，請確認 API Key 是否有效。

---
> Source: [Cheerhuan/save-your-token](https://github.com/Cheerhuan/save-your-token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
