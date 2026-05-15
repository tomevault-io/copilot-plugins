## agent-craft-lab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
dotnet build                                              # 編譯整個 solution

# ─── React 前端 + .NET API 後端 ───
# Terminal 1 — .NET API 後端（port 5200）
dotnet run --project AgentCraftLab.Api

# Terminal 2 — CopilotKit Runtime（port 4000）
cd AgentCraftLab.Web && node server.mjs

# Terminal 3 — React 開發伺服器（port 5173）
cd AgentCraftLab.Web && npm run dev:vite

# 開啟 http://localhost:5173

# MCP 測試 Server（Workflow Studio MCP 工具整合）
npx -y @modelcontextprotocol/server-everything streamableHttp
# → http://localhost:3001/mcp
```

`dotnet test` runs 1446 unit tests. `npx vitest run` runs 456 tests. `npx tsc -b` has 0 errors. `TreatWarningsAsErrors` enabled — style 違規會編譯失敗（`.editorconfig` 規範 file-scoped namespaces、大括號必填、using 排序）。

**整合測試：** `dotnet run --project AgentCraftLab.Autonomous.Playground -- --test`（8 場景，需 Azure OpenAI 憑證）

**API 金鑰：** 前端 `/settings` 設定 → 後端 `ICredentialStore`（DPAPI 加密），前端不持有明文。

## 對談與開發規範

- **使用繁體中文對談**：所有回覆、說明、commit message 皆使用繁體中文
- **善用知識庫與網路搜尋**：開發前可透過 NotebookLM 查詢專案文件，搭配網路搜尋取得最新技術資訊

### 開發流程（強制遵守）

1. **先調研再動手** — 透過 NotebookLM 或 WebSearch 調研，禁止跳過調研直接實作
2. **程式碼 Review** — 完成後執行 `/clean-code-review`，修復所有 Critical/Warning 後才可 commit
3. **補齊單元測試** — 確認新增/修改的 public API 都有測試，`dotnet test` 全過才可 commit

## Critical Constraints

- **禁止 Semantic Kernel**：純用 `Microsoft.Agents.AI` 系列 API（已 GA 1.0.0，csproj 用 `1.*`）
- .NET 10 + LangVersion 13.0
- Schema v2 wire format — `type` discriminator 必須是 JSON 物件第一個屬性（.NET JsonPolymorphic 要求）

## Solution 概覽 — Open Core 架構

| 專案 | 定位 |
|------|------|
| `AgentCraftLab.Api` | .NET Minimal API（AG-UI + REST 端點） |
| `AgentCraftLab.Web` | React 前端（React Flow + CopilotKit + shadcn/ui） |
| `AgentCraftLab.Engine` | 開源核心（5 策略 + 14 節點類型 + 4 層工具 + Middleware + Hooks） |
| `AgentCraftLab.Autonomous` | ReAct 迴圈 + Sub-agent + 16 meta-tools + 安全機制 |
| `AgentCraftLab.Autonomous.Flow` | Flow 結構化執行（LLM 規劃 → 7 節點 → Crystallize） |
| `AgentCraftLab.Cleaner` | 資料清洗（7 格式 Partition + Schema Mapper） |
| `extensions/data/` | 資料層（Data 抽象 + SQLite / MongoDB / PostgreSQL / SqlServer Provider） |
| `extensions/search/` | CraftSearch 搜尋引擎（FTS5 / PgVector / Qdrant / MongoDB / InMemory） |
| `extensions/script/` | 多語言沙箱（Jint JS + Roslyn C#） |
| `extensions/ocr/` | OCR（Tesseract，繁中/簡中/英/日/韓） |

**依賴方向**：`Data`（零依賴）← DB Providers ← `Engine`（只用介面）← `Api`（DI 組合點）

## Architecture 摘要

> 詳細架構說明見 `docs/ARCHITECTURE.md`

- **Workflow Execution**：`WorkflowExecutionService` → Hook → Preprocessor → Strategy → Events。5 種策略（Single/Sequential/Concurrent/Handoff/Imperative），自動偵測
- **節點**：14 種類型 / 12 種可執行（`NodeTypeRegistry` 唯一真相來源），`IsAgentLike` 節點計入 MaxTurns
- **AG-UI 路由**：`/ag-ui`（畫布 Workflow）vs `/ag-ui/goal`（ReAct/Flow 自主模式）
- **Autonomous**：ReactExecutor（策略物件拆分）+ 16 meta-tools（5 層 Tier）+ Checkpoint + 三層記憶 + Multi-Agent Reflexion
- **Flow**：IGoalExecutor 介面隔離，雙模型（Plan gpt-4.1 / Execute gpt-4o-mini），Crystallize 輸出 Studio JSON
- **RAG**：RagService + SearchEngineFactory 多 Provider 路由 + Hybrid 搜尋 + Reranker
- **Middleware**：GuardRails → PII → RateLimit → Retry → Logging
- **Variables**：5 層（sys/node/var/env/runtime），`IVariableResolver` 單一入口
- **Hooks**：6 插入點，CodeHook（TransformKind enum）/ WebhookHook
- **AI Build**：FlowBuilderService → LLM → Schema v2 nested shape，Locale-aware（en/zh-TW/ja）
- **Skills/Tools i18n**：檔案載入（`Data/built-in-skills/` + `Data/built-in-tools/locales/`），三語

## Extensibility（速查）

- **新節點**：(1) `NodeTypes` 常數 (2) `NodeTypeRegistry` 一行 (3) `NodeExecutorRegistry` handler (4) `Schema/Nodes/` record (5) `NodeConfig.cs` `[JsonDerivedType]` (6) JS `NODE_REGISTRY` + `types/workflow.ts` (7) `NodeSpecRegistry`
- **新策略**：實作 `IWorkflowStrategy` + `WorkflowStrategyResolver` 加 case
- **新內建工具**：`ToolImplementations.cs` + `ToolRegistryService.Register()`
- **新 DB Provider**：`extensions/data/` 實作 15 個 Store 介面
- **新 Middleware**：繼承 `DelegatingChatClient` + `ApplyMiddleware()` 加 case
- **新 Flow 節點**：`FlowNodeRunner` + `FlowPlannerPrompt` + `FlowPlanValidator.SupportedNodeTypes` + `WorkflowCrystallizer.StepToNode`
- **替換介面**：`IReflectionEngine` / `IStepEvaluator` / `IScriptEngine` / `IOcrEngine` / `IReranker` / `IPiiDetector` / `IGuardRailsPolicy` — 實作 + DI Replace

## 開源 Repo 同步

**私有**（`F:/codes/AgentFrameworkProject`）→ **開源**（`F:/codes/agent-craft-lab`）

```bash
bash sync-to-opensource.sh "feat: 功能描述"
```

**排除**：`AgentCraftLab.Commercial/` / `AgentCraftLab.CopilotKit/` / `AgentCraftLab.Autonomous.Playground/` / `.claude/` / `nupkgs/`

**新增專案時**：更新 `sync-to-opensource.sh` + `AgentCraftLab.slnx`

## 外部參考資源

- **模型 Context Window 對照表**：`ModelContextWindows.cs`，資料來源 [Azure Foundry Models](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/models-sold-directly-by-azure?tabs=global-standard-aoai%2Cglobal-standard&pivots=azure-openai)
- **設計文件**：`docs/zh-TW/`（繁中）、`docs/en/`（英文）、`docs/ja/`（日文）、`docs/ARCHITECTURE.md`（架構詳細參考）

## Known Limitations

- **A2A auto 模式**：先試 Google 再試 Microsoft 格式，知道對方格式時直接指定可避免延遲
- **AsyncLocal async iterator**：`Activity.Current` yield 後遺失（dotnet/runtime#47802），已用 Event-based TraceCollector workaround

---
> Source: [TimothySu2015/agent-craft-lab](https://github.com/TimothySu2015/agent-craft-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
