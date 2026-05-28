## pdca

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 專案概述

**PDCA** 是一個革命性的 AI 驅動開發系統，透過 5 個並行 AI 代理實現 PDCA（Plan-Do-Check-Act）方法論。系統支援多種 AI 引擎（Claude CLI、Gemini CLI、OpenAI CLI），採用「職人精神」設計理念，追求極致簡潔和優雅體驗。

### 🌟 2025年7月最新特性
- **多 AI 引擎支援**：自動檢測並選擇最佳可用 AI CLI
- **Claude Code Hooks 整合**：利用 PreToolUse、PostToolUse 等鉤子實現智能協調
- **MCP (Model Context Protocol)**：支援外部工具和資源整合
- **真正的並行執行**：基於 tmux 和檔案系統 IPC 的多代理架構

## 🛠️ 常用指令

### 系統執行
```bash
# 方式一：使用 CLI 直接啟動（推薦）
pdca "你的任務描述"

# 方式二：在 Claude Code 中使用斜線指令（僅 Claude CLI）
/pdca:start                    # 啟動多代理系統
/pdca:status                   # 查看運行狀態
/pdca:monitor                  # 開啟監控介面
/pdca:stop                     # 停止系統

# 方式三：指定 AI 引擎
pdca "任務描述" --engine gemini   # 使用 Gemini CLI
pdca "任務描述" --engine claude   # 使用 Claude CLI

# CLI 管理指令
pdca status                    # 查看運行狀態
pdca stop                      # 停止系統
pdca init                      # 初始化專案
pdca doctor                    # 診斷系統環境

# 監控執行中的代理
tmux attach -t pdca
```

### 開發工具（可選）
```bash
# 如需開發或修改系統，可安裝開發工具
pip install black flake8 pytest

# 格式化程式碼
black .

# 執行 linter
flake8 .
```

**注意**：本系統支援多種 AI CLI。若無 Claude CLI，將自動使用 Gemini CLI（免費）或其他可用的 AI 引擎。

### tmux 操作快捷鍵
- **Ctrl+B 1-5**: 切換到代理視窗（plan/do/check/act/knowledge）
- **Ctrl+B 6**: 切換到監控視窗
- **Ctrl+B d**: 分離 session（背景執行）

## 🏗️ 系統架構

### 多代理系統
系統並行啟動 5 個獨立的 AI 代理實例：
1. **pdca-plan**: 需求分析、策略制定、任務協調
2. **pdca-do**: 架構設計、功能實作、代碼開發
3. **pdca-check**: 品質驗證、測試檢查、結果評估
4. **pdca-act**: 性能優化、問題改善、持續改進
5. **knowledge-agent**: 智能監聽、分類歸檔、經驗累積

### 核心技術
- **tmux**: 管理並行代理 sessions
- **git worktree**: 為每個代理提供隔離工作空間（可選）
- **檔案系統 IPC**: 代理透過 `.raiy-pdca/communication/` 通訊
- **AI 引擎適配器**: 統一支援 Claude、Gemini、OpenAI 等 CLI
- **TUI 監控**: 即時狀態監控介面

### 目錄結構
```
.raiy-pdca/             # 核心系統（包含啟動器和監控）
├── launcher.py         # 主要進入點 - 創建 tmux、啟動代理
├── monitor.py          # TUI 監控介面 - 實時顯示代理狀態
├── agents/             # 代理配置（執行時建立）
├── worktrees/          # 隔離的 git 工作空間（執行時建立）
├── communication/      # 代理間訊息傳遞（執行時建立）
└── logs/               # 系統日誌（執行時建立）

memories/               # 知識庫
├── decisions/          # 決策記錄
├── solutions/          # 解決方案
├── patterns/           # 設計模式
├── learnings/          # 經驗教訓
└── progress/           # 進度追蹤

pdca_shokunin/          # Python 套件目錄（舊版本遺留，不影響運行）
```

## 🎯 開發準則

### 語言與溝通
- **主要語言**: 中文用於使用者介面和文檔
- **技術術語**: 保持英文（如 API、REST、tmux）
- **註解**: 中文說明 WHY/HOW，技術術語保留英文

### 程式碼風格
- **Python**: 遵循 PEP 8，公開函式加上 type hints
- **格式化**: 使用 `black`（在 pyproject.toml 中配置）
- **Linting**: 使用 `flake8`（無設定檔，使用預設值）

### Git 工作流程
- **原子化提交**: 每次只改一個功能
- **提交訊息**: 遵循格式 `[YYYYMMDD] type: 描述`
- **禁止詞彙**: 絕不在提交中使用 AI/Claude/Generated/Bot

### 測試策略
- 目前專案中無測試檔案
- 新增測試時使用 `pytest` 框架
- 測試放在 `tests/` 目錄或程式碼旁的 `test_*.py`

## ⚠️ 重要提醒

1. **系統特性**: 智能多 AI 引擎支援，自動選擇最佳可用 CLI
2. **依賴需求**: Node.js 16+、tmux、git、至少一個 AI CLI
3. **支援的 AI CLI**: 
   - Claude CLI（需訂閱）
   - Gemini CLI（免費，推薦）
   - OpenAI CLI（需 API key）
4. **執行方式**: 使用 `pdca` 命令或 Claude Code 中的 `/pdca` 斜線指令
5. **中文優先**: 所有使用者介面內容應優先使用中文

## 🚀 快速任務

### 新增功能（使用 CLI）
1. 啟動 PDCA 系統：`pdca "新增功能 X"`
2. 監控進度：`tmux attach -t pdca`
3. 代理會自動協調工作

### 使用 Claude Code 斜線指令
1. 開啟 Claude Code：`claude`
2. 啟動系統：`/pdca:start`
3. 告訴我任務：例如「實作使用者登入功能」
4. 查看狀態：`/pdca:status`
5. 監控進度：`/pdca:monitor`

### 除錯問題
1. 檢查代理日誌：`.raiy-pdca/logs/`
2. 查看通訊內容：`.raiy-pdca/communication/`
3. 使用監控介面查看即時狀態

### 知識管理
- 決策記錄放在 `memories/decisions/`
- 解決方案放在 `memories/solutions/`
- 設計模式放在 `memories/patterns/`
- 所有記憶檔案應使用 Markdown 格式並以中文撰寫

## 🔄 記憶系統規範（強制）

### 記憶檔案結構
1. **長期記憶**：`memories/long_term/`，檔名格式 `YYYYMMDD_<slug>.md`
2. **短期記憶**：`memories/short_term/last_session.md`

### 讀取流程（每次對話開始時）
1. 載入 `last_session.md`
2. 拼接必要的長期記憶背景
3. 生成上下文摘要

### 寫入規則
- **重要業務決策** → 新增至 `memories/long_term/`
- **流程中的臨時狀態** → 更新 `memories/short_term/last_session.md`

### 同步提醒
- 完成重要工作後，在回覆結尾加上 `🔄` 提醒記憶檔案待同步
- 同步順序：`CLAUDE.md` → `docs/` → `memories/`

## 🔴 Git 提交規範（必須遵守）

### 重要提醒
**每次完成一段工作後必須進行 git commit**。這是強制性要求，不可省略。

### 提交檢查清單
在完成任何功能修改後：
1. 執行 `git status` 查看變更
2. 執行 `git add .` 加入變更
3. 使用正確格式提交：`[YYYYMMDD] type: 描述`
4. 確保無禁用詞彙（AI、Claude、Generated 等）
5. 提交訊息長度 ≤ 50 字元

### 提交時機
- ✅ 完成一個功能模組後
- ✅ 修正一個錯誤後
- ✅ 更新配置檔案後
- ✅ 重構程式碼後
- ❌ 不要累積過多變更才提交

### 範例
```bash
git add .
git commit -m "[20250709] feat: 實作多 AI 引擎支援"
```

## 🔧 多 AI 引擎實現方案

### Claude CLI 模式
- **特點**: 支援斜線指令、完整檔案操作、目錄感知
- **啟動方式**: 互動式 session，無法直接執行 prompt
- **適用場景**: 需要複雜檔案操作的開發任務

### Gemini CLI 模式（推薦）
- **特點**: 支援 `-p` 參數直接執行、免費使用、60 req/min
- **啟動方式**: `gemini -p "prompt"` 一次性執行
- **適用場景**: 大部分開發任務，特別適合自動化流程

### 混合模式
系統自動檢測可用的 AI CLI，優先順序：
1. Claude CLI（如已安裝且配置）
2. Gemini CLI（免費且功能完整）
3. 其他支援的 AI CLI

## 📡 Claude Code Hooks 整合

### 可用的 Hook 類型
- **PreToolUse**: 工具使用前執行，可阻擋或修改操作
- **PostToolUse**: 工具使用後執行，記錄結果
- **Notification**: 系統通知時觸發
- **Stop**: 主代理完成回應時執行

### Hook 配置範例
```json
{
  "hooks": {
    "matchers": [{
      "tools": ["Bash", "Edit"],
      "preToolUse": "echo 'Tool: {{tool_name}}' >> .pdca/activity.log"
    }]
  }
}
```

## 🌐 MCP (Model Context Protocol)

### 概述
MCP 是 Anthropic 於 2024 年 11 月推出的開放協議，讓 LLM 能夠存取外部工具和數據源。

### 整合方式
```bash
# 添加 MCP 伺服器
claude mcp add pdca-tools "node pdca-mcp-server.js"

# 使用 @ 符號引用資源
@pdca-status
@pdca-logs
```

## 📚 技術參考資源

### 社群方案
1. **Claude Squad** (https://github.com/smtg-ai/claude-squad)
   - 使用 tmux + git worktrees 管理多個 AI 終端代理
   - 支援 Claude Code、Aider、Codex 等

2. **Claude Flow v2.0.0** (https://github.com/ruvnet/claude-flow)
   - Queen AI 協調系統，87+ MCP 工具
   - 企業級 AI 群體協調平台

3. **VibeTunnel** (https://github.com/amantus-ai/vibetunnel)
   - 遠端終端監控工具
   - 可透過瀏覽器控制 AI 代理

### 官方文檔
- Claude Code Hooks: https://docs.anthropic.com/en/docs/claude-code/hooks
- MCP Protocol: https://docs.anthropic.com/en/docs/claude-code/mcp
- Gemini CLI: https://cloud.google.com/gemini/docs/gemini-cli

### 相關工具
- tmux: 終端多路復用器
- git worktree: Git 工作樹隔離
- GNU parallel: 並行執行工具

---
> Source: [RayYangTW/pdca](https://github.com/RayYangTW/pdca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
