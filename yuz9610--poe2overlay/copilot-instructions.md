## poe2overlay

> > **人類開發者**請改讀 [docs/README.md](docs/README.md) → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。本頁供 Codex / Cursor 等工具自動載入。

# AI Agent 指南

> **人類開發者**請改讀 [docs/README.md](docs/README.md) → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。本頁供 Codex / Cursor 等工具自動載入。  
> **Workflow 人類速查**（短命令、跳過確認、恢復 session）：[docs/AI_AGENT_SETUP.md](docs/AI_AGENT_SETUP.md)。  
> **Claude / Claude Code 入口**：根目錄 [CLAUDE.md](CLAUDE.md)（co-author: `ClaudeAgent`）。  
> **Kiro 入口**：[`.kiro/steering/`](.kiro/steering/)（co-author: `KiroAgent`）。

本 repo 的 **文檔目標架構** 在 [`docs/README.md`](docs/README.md#目標信息架構)；**技術總覽** 在 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)。開始改碼前請先讀對應文件；本頁為精簡入口，不承載長篇規則。

## 必讀

1. **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)** — 技術棧、目錄、資料流、IPC 表、常見修改入口
2. **[docs/ENCODING.md](docs/ENCODING.md)** — UTF-8 無 BOM、LF、pwsh 7+；含中文 JSON 勿改編碼
3. **[docs/PROJECT_WORKFLOW.md](docs/PROJECT_WORKFLOW.md)** — **workflow 單一真相**（gate、狀態轉移、commit、diagnostic log）
4. **[docs/AI_AGENT_PROMPTS.md](docs/AI_AGENT_PROMPTS.md)** — 六步 prompt 模板（規則以 PROJECT_WORKFLOW 為準）

## 文檔信息架構（agent 必須遵守）

完整表格見 **[docs/README.md § 目標信息架構](docs/README.md#目標信息架構)**。修改文件時先判斷知識類型，只更新對應單一真相來源（SOT），其他入口只放短摘要與連結。

| 知識類型 | 單一真相來源 |
|----------|--------------|
| 技術架構、IPC、資料流、模組 | `docs/ARCHITECTURE.md` |
| 玩家操作與可見 UI | `docs/USER_GUIDE.md` |
| Build JSON schema / 支援範圍 | `docs/BUILD_JSON.md` |
| 由攻略生成 Build JSON 的流程 | `docs/CREATE_BUILD.md` |
| 編碼、LF、BOM、PowerShell 政策 | `docs/ENCODING.md` |
| idea / sprint / task / accept / commit 流程 | `docs/PROJECT_WORKFLOW.md` |
| Prompt 模板 | `docs/AI_AGENT_PROMPTS.md`（不得新增獨有流程規則） |
| Agent 入口維護 | `docs/AI_AGENT_SETUP.md` |
| 當前 idea / sprint 狀態 | `docs/BRAINSTORM.md`、`docs/SPRINTS.md`、`docs/sprints/` |

**新增或修改 docs 時**：每個 `docs/**/*.md` 頂部都要有 `文檔目的`／`不負責` 定位；避免把同一規則長篇複製到多處。`AGENTS.md` 與 `.cursor/rules/*.mdc` 只做 agent 可見摘要。

## 專案一句話

Tauri 2 桌面：**PoE2 Area Overlay**（Rust 讀 `Client.txt` → 區域／BD 卡片；build JSON **只讀**）+ 規劃中 **PoE2 BD Manager**（第二 exe，編輯 `data/build/`）。全部本機，無 HTTP。

## 關鍵路徑

| 用途 | 路徑 |
|------|------|
| IPC 前端入口 | `src/api/tauri.ts` |
| 應用狀態 | `src/app/useOverlayData.ts` |
| Tauri 命令 | `src-tauri/src/lib.rs` |
| 日誌監聽 | `src-tauri/src/watcher.rs`、`log_parser.rs` |
| 區域提示 | `src-tauri/src/area_hints.rs`、`data/areas.json` |
| BD 資料（Overlay 只讀） | `docs/BUILD_JSON.md`、`data/build/` |
| Sprint / task | `docs/SPRINTS.md`、`docs/sprints/` |
| 型別 | `src/types.ts` |

## Workflow 五條硬規則

完整說明、狀態轉移表、commit 格式見 **[docs/PROJECT_WORKFLOW.md § Agent 強制規則](docs/PROJECT_WORKFLOW.md#agent-強制規則)**。短命令與範例見 **[docs/AI_AGENT_PROMPTS.md](docs/AI_AGENT_PROMPTS.md)**。

1. **先 proposal，後寫回** — workflow 短命令第一次回覆不得改 repo、不得 commit（可讀檔探索）。
2. **跳過確認** — 僅當使用者**同一則訊息**明確說 `跳过确认` / `跳過確認` / `skip confirmation` / `直接执行` / `直接修改` 等；`快点做`、`In progress` **不算**。
3. **`執行 task` = 兩階段** — 第一次只出 execution plan；確認後才實作；**Step 4 不 commit**。
4. **`驗收 task` = 先 review** — 第一次只给 Accept / Needs changes / Blocked；agent **自行读** `logs/*.log` 对照 task doc；人类确认 Accept 后才标 Done、downgrade log、commit。
5. **改 workflow 規則** — 先改 `PROJECT_WORKFLOW.md`，再同步 `AI_AGENT_PROMPTS.md` 開頭、`AI_AGENT_SETUP.md`、本檔（`AGENTS.md`）、`CLAUDE.md`、`.cursor/rules/workflow.mdc`、`.kiro/steering/workflow.md`。

| 短命令 | 第一次回覆 |
|--------|------------|
| `brainstorm` / `review` / `sprint planning` | proposal |
| `執行 task Ixxx-yy` | 階段 A plan |
| `驗收 task Ixxx-yy` | acceptance review |
| `Sprint X summary` | summary proposal（含 workflow / project / docs review、SPRINTS.md status / ordering writeback） |

Step 5 commit：第一行 `Ixxx Item Name Ixxx-yy Accept task`；body + `Co-authored-by: CursorAgent`；Windows 用 `git commit -F`（UTF-8 無 BOM）。見 PROJECT_WORKFLOW § Commit Message。

Step 6 summary 確認寫回時，必須同步 `SPRINTS.md` Current Snapshot、parent item status、Task List status / sprint 欄位，並重排 Above The Line Items / Task List，使兩表 item 順序一致（進行中 / 已開始、未開始、完成）。

## 開發命令

```pwsh
npm.cmd run format
npm.cmd run check:format
npm.cmd run check:encoding
npm.cmd run build
npm.cmd run tauri -- dev
cd src-tauri; cargo clippy -- -D warnings
```

Windows：用 **pwsh** 與 **npm.cmd**。Shell 優先 `pwsh` 7+，勿用 Windows PowerShell 5.1。

## 其他硬性規則

- 所有文字檔 **UTF-8 無 BOM**，換行 **LF**（`.bat`/`.cmd` 除外）
- `data/areas.json` 區域名必須與 `Client.txt` `[SCENE] Set Source […]` **位元組一致**
- 新 Tauri 能力：同時改 Rust、`src/api/tauri.ts`、必要時 `src/types.ts`；勿在 Rust 加 shell／網路
- 功能／行為變更：同改動內更新文檔，矩陣見 [ARCHITECTURE § 文檔同步](docs/ARCHITECTURE.md#文檔同步)
- 文檔結構／新增 docs：同步 [docs/README.md § 目標信息架構](docs/README.md#目標信息架構)，並在新文檔頂部寫清 `文檔目的`／`不負責`

## 其他文檔

- 玩家操作：[docs/USER_GUIDE.md](docs/USER_GUIDE.md)
- 入門與打包：[README.md](README.md)

---
> Source: [yuz9610/POE2Overlay](https://github.com/yuz9610/POE2Overlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
