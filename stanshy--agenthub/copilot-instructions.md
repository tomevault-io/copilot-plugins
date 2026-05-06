## agenthub

> > 本文件是地圖，不是百科全書。詳細規範見索引指向的 .knowledge/ 文件。

# AgentHub — Agent 指南（索引）

> 本文件是地圖，不是百科全書。詳細規範見索引指向的 .knowledge/ 文件。
> **所有 Agent 執行任務前必須先閱讀本文件，再依索引查閱對應文件。**

> 🚀 **首次 clone 後請執行**：`bash scripts/install-skills.sh`（將公司級 Skills 安裝到全域 `~/.claude/commands/`）

---

## 專案簡介

- **專案名稱**: AgentHub (Maestro v2)
- **類型**: Electron 桌面應用（AI Agent 團隊管理平台）
- **核心價值**: 用 Harness（Skill + Hook + FileWatcher）驅動虛擬開發公司，GUI 只做監控和 3 個操作
- **目標用戶**: 老闆（一人公司創辦人）
- **開發平台**: Windows 11

---

## 最高原則（3 條致命規則）

1. **文件就是法律** — 程式碼必須與規範文件一致，不一致以文件為準。
   → 完整 8 條規則見 `.knowledge/doc-governance.md`

2. **IPC 四方同步** — `ipc.ts` → `preload.ts` → `useIpc.ts` → `env.d.ts` 缺一不可。
   → 詳見 `.knowledge/architecture.md`「IPC 三方一致規則」

3. **不確定就去讀，不要猜** — 不得憑空想像任何資料結構或 API 格式。
   → 真相來源表見 `.knowledge/coding-standards.md`「禁止憑空想像規則」

> ※ 危險指令已由 PreToolUse Hook 強制攔截（kill-port / --no-verify / force push main），見 `.claude/settings.json`

---

## 常用指令

```bash
npm run dev          # 啟動開發模式
npm run test         # 單元測試
npm run lint         # ESLint 檢查
npm run typecheck    # TypeScript 型別檢查
npm run build        # 打包
```

---

## 專案文件索引

### 專案級規範（.knowledge/）

| 文件 | 用途 | 版本 |
|------|------|------|
| `.knowledge/project-overview.md` | 專案概述、目標、技術棧、v1→v2 變更摘要 | v1.1 |
| `.knowledge/architecture.md` | 系統架構、服務清單、IPC 架構、三方一致規則 | v1.1 |
| `.knowledge/directory-structure.md` | 目錄結構詳細說明 | v1.0 |
| `.knowledge/coding-standards.md` | 編碼規範、命名、Commit 紀律、依賴規則、禁止憑空想像 | v1.1 |
| `.knowledge/testing-standards.md` | 測試策略與規範 | v1.0 |
| `.knowledge/quality-checklist.md` | G0-G6 品質檢查清單 | v1.0 |
| `.knowledge/postmortem-log.md` | 踩坑快速參考 + 歷史紀錄 | v1.1 |
| `.knowledge/doc-governance.md` | 文件治理 8 條完整規則 | v1.0 |
| `.knowledge/team-hierarchy.md` | 團隊架構、指揮鏈、Sprint 概覽 | v1.0 |

### 公司規範（.knowledge/company/）

| 文件 | 用途 |
|------|------|
| `.knowledge/company/sop/sprint-planning.md` | Sprint 規劃 SOP v4.1 |
| `.knowledge/company/sop/code-review.md` | 程式碼審查 SOP |
| `.knowledge/company/standards/coding-standards.md` | 公司編碼標準 |
| `.knowledge/company/standards/api-standards.md` | 公司 API 標準 |
| `.knowledge/company/standards/quality-checklist.md` | 公司品質檢查清單 |
| `.knowledge/company/standards/project-rules.md` | 子專案共用開發規則 |
| `.knowledge/company/standards/team-workflow.md` | 子專案共用團隊流程 |
| `.knowledge/company/templates/sprint-proposal.md.template` | Sprint 提案書範本 |
| `.knowledge/company/templates/dev-plan.md.template` | 開發計畫書範本 |
| `.knowledge/company/templates/internal-review.md.template` | 內部審查報告範本 |

---
> Source: [Stanshy/AgentHub](https://github.com/Stanshy/AgentHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
