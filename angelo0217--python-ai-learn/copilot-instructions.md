## python-ai-learn

> 這個專案 (`python_ai_learn`) 是一個基於 Python 的 AI/LLM 學習與實驗儲存庫。它包含了多個獨立的模組，用於探索不同的 AI 代理 (Agent) 框架、大型語言模型 (LLM) 的應用、程式碼生成、以及檢索增強生成 (RAG) 技術。

# Gemini Project Overview: python_ai_learn

## 1. 專案總覽

這個專案 (`python_ai_learn`) 是一個基於 Python 的 AI/LLM 學習與實驗儲存庫。它包含了多個獨立的模組，用於探索不同的 AI 代理 (Agent) 框架、大型語言模型 (LLM) 的應用、程式碼生成、以及檢索增強生成 (RAG) 技術。

此專案並非單一應用程式，而是一個包含各種範例和展示的程式碼集合。

## 2. 關鍵技術與框架

- **程式語言:** Python
- **套件與環境管理:** `uv` (根據 `uv.lock` 和 `.python-version` 檔案判斷)
- **主要 AI/LLM 框架:**
    - **LangChain:** 用於建構 LLM 應用，特別是多代理人協作 (Group/Graph Chats) 和 RAG。
    - **AutoGen:** 用於建立多代理人對話系統。
- **使用的模型/API:**
    - Google Gemini (`gen_code_gemini`)
    - Ollama (`auto_gen/ollama_auto_gen.py`)
    - OpenAI (推斷自 `.venv` 中的 `openai.exe`)
- **資料庫/向量儲存:**
    - Redis (用於 LangChain RAG，如 `langchain_demo/redis_rag.py`)
- **程式碼格式化:**
    - `black` (推斷自 `.venv` 中的 `black.exe`)

## 3. 目錄結構

- **`/auto_gen/`**: 包含使用 Microsoft AutoGen 框架的實驗。涵蓋了代理人群組、與 Jira 的整合、以及使用 Ollama 本地模型的範例。
- **`/gen_code_gemini/`**: 專注於使用 Google Gemini 模型進行程式碼生成的範例。
- **`/gen_code_xai/`**: 似乎是另一個程式碼生成與評估的模組，可能與 "XAI" (Explainable AI) 或某個特定模型/平台相關。
- **`/langchain_demo/`**: 包含使用 LangChain 框架的各種範例，如 RAG、多代理人綁定 RAG、以及圖形化的代理人對話流程。
- **`/mcp_demo/`**: 包含名為 "mcp" 的範例，可能是一個自訂的代理人或框架模式。範例涉及 SQL 操作和即時匯率查詢，並使用 SSE (Server-Sent Events)。
- **`main.py`**: 可能是專案的主要進入點或一個測試/範例腳本。
- **`pyproject.toml`**: Python 專案的設定檔，定義了專案元數據和依賴項。
- **`uv.lock`**: `uv` 套件管理器的鎖定檔案，確保依賴項版本的一致性。

## 4. 開發與執行指令

- **安裝/同步依賴:**
  ```bash
  uv sync
  ```
- **執行範例:**
  各個目錄下的 Python 腳本是獨立的，可以直接執行。例如：
  ```bash
  python langchain_demo/rag_api.py
  ```
- **程式碼格式化:**
  ```bash
  black . -l 100
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/angelo0217)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/angelo0217)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
