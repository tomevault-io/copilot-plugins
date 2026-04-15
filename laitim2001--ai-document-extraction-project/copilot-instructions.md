## ai-document-extraction-project

> > 本文件為 Claude Code AI 助手的開發指引，自動作為 system prompt 載入。

# AI Document Extraction Project - Claude Code 開發指引

> 本文件為 Claude Code AI 助手的開發指引，自動作為 system prompt 載入。
> 詳細規範已拆分至 `.claude/rules/` 和 `claudedocs/reference/`，本文件保留核心摘要與引用路徑。

---

## 🗣️ 語言設定（🔴 必須遵守）

> **核心規則**: AI 助手在所有對話中**必須全程使用繁體中文**回應用戶。這是最高優先級的溝通規則，無論任何情境、任何技術討論，都不可使用英文或其他語言回覆用戶。技術術語和代碼標識符保留原文即可。

| 場景 | 語言 | 強制等級 |
|------|------|----------|
| **用戶對話與回應** | 繁體中文 | 🔴 必須（無例外） |
| **問題解釋與建議** | 繁體中文 | 🔴 必須（無例外） |
| **代碼註釋** | 中文或英文 | 🟡 依上下文選擇 |
| **Commit Message** | 英文 | 🟡 遵循 Conventional Commits |
| **技術文檔** | 繁體中文為主 | 🟡 claudedocs/ 和 docs/ |
| **代碼中的 UI 字串** | 使用 i18n key | 🔴 禁止硬編碼 |

---

## 項目概覽

### 項目使命

建立一套 AI 驅動的文件內容提取與自動分類系統，專門解決 SCM 部門處理 Freight Invoice 的效率問題。

### 核心目標

- **年處理量**: 450,000-500,000 張發票（APAC 地區）
- **自動化率**: 90-95%
- **準確率**: 90-95%
- **節省人時**: 35,000-40,000 人時/年

### 核心架構 - 三層映射系統

```
┌─────────────────────────────────────────────────────────────────┐
│ TIER 1: Universal Mapping（通用層）                              │
│ • 覆蓋 70-80% 常見術語，所有 Forwarder 通用                      │
│ • 維護成本：低（只需維護一份）                                   │
├─────────────────────────────────────────────────────────────────┤
│ TIER 2: Forwarder-Specific Override（特定覆蓋層）                │
│ • 只記錄該 Forwarder 與通用規則「不同」的映射                    │
│ • 維護成本：中（每個 Forwarder 只需記錄差異）                    │
├─────────────────────────────────────────────────────────────────┤
│ TIER 3: LLM Classification（AI 智能分類）                        │
│ • 當以上都無法匹配時，使用 GPT-5.2 智能分類                      │
│ • 可處理從未見過的新術語                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 信心度路由機制

| 信心度範圍 | 處理方式     | 說明                          |
| ---------- | ------------ | ----------------------------- |
| ≥ 95%      | AUTO_APPROVE | 自動通過，無需人工介入        |
| 80-94%     | QUICK_REVIEW | 快速人工確認（一鍵確認/修正） |
| < 80%      | FULL_REVIEW  | 完整人工審核（詳細檢查）      |

> **V3.1 智能降級**: 新公司 → 強制 FULL_REVIEW；新格式 → 強制 QUICK_REVIEW；DEFAULT 配置來源 → 降一級路由。

---

## 技術棧

### 核心框架

- **前端**: Next.js 15.0.0 (App Router) + TypeScript 5.0 + React 18.3
- **樣式**: Tailwind CSS 3.4 + shadcn/ui (Radix UI 20+ primitives)
- **資料庫**: PostgreSQL 15 + Prisma ORM 7.2 (117 models)
- **狀態管理**: Zustand 5.x (UI) + React Query 5.x (Server State)
- **表單**: React Hook Form 7.x + Zod 4.x 驗證
- **國際化**: next-intl 4.7 (支援 en, zh-TW, zh-CN，30 個命名空間)
- **快取**: @upstash/redis
- **拖放**: @dnd-kit (sortable UI)

### 外部服務

- **OCR**: Azure Document Intelligence
- **AI**: Azure OpenAI GPT-5.2 + OpenAI SDK
- **認證**: Azure AD (Entra ID) SSO + 本地帳號認證
- **工作流**: n8n
- **文件來源**: SharePoint / Outlook / Azure Blob Storage
- **Office 365**: Microsoft Graph Client

### 文件處理

- **PDF**: pdfjs-dist, pdf-parse, pdf-to-img, react-pdf, pdfkit
- **Excel**: ExcelJS (匯出功能)
- **Email**: Nodemailer (通知系統)

### 開發工具

- **容器化**: Docker Compose (PostgreSQL + pgAdmin + Azurite + Python OCR/Mapping)
- **代碼品質**: ESLint + Prettier
- **E2E 測試**: Playwright 1.57
- **套件管理**: npm

### 代碼規模概覽

| 指標 | 數量 | 說明 |
|------|------|------|
| React 組件 | 165+ | `src/components/` 下所有 `.tsx` |
| 業務服務 | 124+ | `src/services/` 下所有 `.ts` |
| API 路由文件 | 175+ | 每個 route.ts 處理多個 HTTP 方法，約 300+ 端點 |
| 自定義 Hooks | 89 | `src/hooks/` |
| Prisma Models | 117 | 資料庫模型定義 |
| i18n 命名空間 | 30/語言 | 3 種語言各 30 個 JSON 文件 |

---

## 目錄結構（精簡版）

```
ai-document-extraction-project/
├── .claude/              # Claude Code 配置（rules/ 9 規則 + agents/ 8 個 + skills/ 4 個）
├── claudedocs/           # AI 助手文檔（planning/sprints/progress/changes/status/reference）
├── docs/                 # 項目正式文檔（discovery/planning/architecture/stories/implementation）
├── messages/             # i18n 翻譯文件（en/ zh-TW/ zh-CN/，各 30 個命名空間）
├── prisma/               # Prisma Schema（117 models）和遷移
├── python-services/      # Python 後端（extraction/ + mapping/）
├── src/
│   ├── app/[locale]/     # App Router（(auth)/ + (dashboard)/ 25+ admin 頁面）
│   ├── app/api/          # API 路由（175 files, ~300 端點）
│   ├── components/       # React 組件（ui/ + features/ + layout/，165+）
│   ├── hooks/            # 自定義 Hooks（89 個）
│   ├── i18n/             # i18n 配置（config.ts + routing.ts + request.ts）
│   ├── lib/              # 工具庫（含 validations/ Zod Schema）
│   ├── services/         # 業務邏輯服務層（124+，含 extraction-v3/ 三階段管線）
│   ├── stores/           # Zustand 狀態管理
│   └── types/            # TypeScript 類型定義
└── tests/                # 測試（unit/ + integration/ + e2e/）
```

> 📋 完整目錄結構（含 API 路由、Agents/Skills/Rules 詳表）：`claudedocs/reference/directory-structure.md`

---

## ClaudeDocs - AI 助手文檔目錄

> **完整索引**: `claudedocs/CLAUDE.md`

| 用途            | 目錄              | 說明                     |
| --------------- | ----------------- | ------------------------ |
| **規劃文檔**    | `1-planning/`     | Epic 架構設計、功能規劃  |
| **Sprint 追蹤** | `2-sprints/`      | Sprint 計劃記錄          |
| **進度報告**    | `3-progress/`     | 日報/週報、進度追蹤      |
| **變更管理**    | `4-changes/`      | Bug 修復、功能變更記錄   |
| **狀態報告**    | `5-status/`       | 測試報告、系統狀態       |
| **AI 協作**     | `6-ai-assistant/` | 情境提示詞、工作流程指南 |
| **歸檔**        | `7-archive/`      | 舊版文檔、範本           |
| **參考資料**    | `reference/`      | 目錄結構、進度表、檢查清單 |

### AI 助手情境提示詞 (SITUATION)

| 情境            | 文件                     | 使用時機             |
| --------------- | ------------------------ | -------------------- |
| **SITUATION-1** | `PROJECT-ONBOARDING.md`  | 新會話啟動、項目入門 |
| **SITUATION-2** | `FEATURE-DEV-PREP.md`    | 功能開發前的任務分析 |
| **SITUATION-3** | `FEATURE-ENHANCEMENT.md` | 功能增強、代碼優化   |
| **SITUATION-4** | `NEW-FEATURE-DEV.md`     | 新功能實作執行       |
| **SITUATION-5** | `SAVE-PROGRESS.md`       | 保存進度、會話結束   |
| **SITUATION-6** | `SERVICE-STARTUP.md`     | 服務啟動、環境重啟   |
| **SITUATION-7** | `SEED-DATA-MAINTENANCE.md` | Seed 數據維護、部署 |

**路徑**: `claudedocs/6-ai-assistant/prompts/SITUATION-*.md`

---

## 代碼規範（摘要）

### 文件頭部註釋（必須遵守）

所有業務邏輯文件必須包含標準 JSDoc 頭部註釋（含 `@fileoverview`、`@module`、`@since`、`@lastModified`）。

| 文件類型                    | 是否需要完整頭部 | 說明           |
| --------------------------- | ---------------- | -------------- |
| Services (`src/services/`)  | ✅ 必須          | 核心業務邏輯   |
| API Routes (`src/app/api/`) | ✅ 必須          | API 端點       |
| Hooks (`src/hooks/`)        | ✅ 必須          | 自定義 Hooks   |
| Utils (`src/lib/`)          | ✅ 必須          | 工具函數       |
| Feature Components          | ✅ 必須          | 功能性組件     |
| UI Components (`ui/`)       | ❌ 不需要        | shadcn/ui 組件 |
| Type Definitions            | ⚠️ 簡化版        | 類型文件       |

### 命名規範

| 類型      | 規範             | 範例                       |
| --------- | ---------------- | -------------------------- |
| 文件名    | kebab-case       | `use-document-upload.ts`   |
| 組件      | PascalCase       | `DocumentUploadForm.tsx`   |
| 函數      | camelCase        | `calculateConfidenceScore` |
| 常數      | UPPER_SNAKE_CASE | `MAX_FILE_SIZE`            |
| 類型/介面 | PascalCase       | `DocumentMetadata`         |
| Enum 值   | UPPER_SNAKE_CASE | `Status.PENDING_REVIEW`    |

> 📋 JSDoc 完整模板與代碼範例：`.claude/rules/general.md`
> 📋 TypeScript 完整規範（類型定義、嚴格模式）：`.claude/rules/typescript.md`

---

## API 設計規範（摘要）

- **成功響應**: `{ success: true, data: T, meta?: { pagination? } }`
- **錯誤響應**: RFC 7807 格式 `{ type, title, status, detail, instance, errors? }`
- **路由總量**: 175 route files，約 300+ 端點

> 📋 完整 API 設計規範與代碼範例：`.claude/rules/api-design.md`
> 📋 API 路由完整目錄結構：`claudedocs/reference/directory-structure.md`

---

## 資料庫規範（摘要）

- **ID 策略**: 新建模型一律使用 `@default(uuid())`。舊模型部分仍使用 `cuid()`，正在逐步遷移中
- **遷移命名**: `YYYYMMDDHHMMSS_描述`（如 `npx prisma migrate dev --name add_confidence_score`）
- **Schema 命名**: 模型 PascalCase 單數、欄位 camelCase、表名 snake_case 複數

> 📋 完整資料庫規範與 Schema 範例：`.claude/rules/database.md`

---

## 組件與狀態管理（摘要）

- **UI 狀態**: Zustand（`src/stores/`）— 側邊欄、對話框等 UI 交互狀態
- **伺服器狀態**: React Query（`src/hooks/`）— API 數據獲取、快取、同步
- **組件結構**: `'use client'` → imports → interface → component function → return JSX

> 📋 完整組件開發規範與代碼範例：`.claude/rules/components.md`

---

## 測試規範（摘要）

| 類型 | 覆蓋率目標 | 範圍 |
|------|-----------|------|
| 單元測試 | ≥ 80% | 核心業務邏輯 |
| 整合測試 | ≥ 70% | API 端點 |
| E2E 測試 | 關鍵流程 | 文件上傳、審核流程 |

> 📋 完整測試規範與目錄結構：`.claude/rules/testing.md`

---

## Git 規範

### 分支命名

```
main                                  # 生產分支
feature/change-NNN-description        # 功能變更分支（當前主要使用）
fix/issue-description                 # 修復分支
hotfix/critical-issue                 # 緊急修復
```

> 項目已從 `feature/epic-X-story-Y` 演進到 `feature/change-NNN` 模式。

### Commit Message 格式

```
<type>(<scope>): <subject>
```

**類型**: `feat` | `fix` | `docs` | `style` | `refactor` | `test` | `chore`

---

## 開發工作流

### 快速啟動

```bash
docker-compose up -d          # 1. 啟動 Docker 服務（PostgreSQL + Azurite）
npm install                   # 2. 安裝依賴
npx prisma generate           # 3. 生成 Prisma Client
npx prisma migrate dev        # 4. 執行遷移
npm run dev                   # 5. 啟動開發伺服器（預設 port 3000）
npm run dev -- -p 3200        # 5b. 備用端口（推薦：3200/3300/3500）
```

### 代碼提交前檢查

```bash
npm run type-check            # TypeScript 類型檢查
npm run lint                  # ESLint 檢查
npm run format                # Prettier 格式化
npm run test                  # 測試
```

> 📋 詳細服務啟動流程與問題排解：`.claude/CLAUDE.md`

---

## AI 開發輔助指引

### 優先參考文檔

1. **Tech Specs**: `docs/03-stories/tech-specs/` - 各 Story 的技術規格
2. **Implementation Context**: `docs/04-implementation/implementation-context.md`
3. **PRD**: `docs/01-planning/prd/prd.md`

### 代碼生成規則

1. **必須添加標準註釋** - 按上述規範添加文件頭部和函數註釋
2. **遵循現有模式** - 參考 `src/` 下現有代碼風格
3. **類型安全優先** - 使用嚴格的 TypeScript 類型
4. **Zod 驗證** - API 輸入必須使用 Zod schema 驗證
5. **錯誤處理** - 使用 RFC 7807 格式返回錯誤

### 禁止事項

- ❌ 不要使用 `any` 類型
- ❌ 不要跳過錯誤處理
- ❌ 不要硬編碼敏感資訊
- ❌ 不要在客戶端組件中直接訪問資料庫
- ❌ 不要忽略 TypeScript 錯誤
- ❌ **不要擅自偏離設計規格**（見技術障礙處理規範）

---

## 🤖 開發編排協議 — 並行 Agent 自動化（🟡 重要）

> **核心原則**: 當任務可拆分為多個獨立子任務時，AI 助手應**主動**使用並行 Agent 編排來最大化開發效率。不需要用戶明確指示。

### 自動觸發條件

| 條件 | 說明 | 範例 |
|------|------|------|
| **多 CHANGE/FIX 同時開發** | 2+ 個獨立的 CHANGE 或 FIX | Sprint 含 CHANGE-043 + CHANGE-044 |
| **大型功能實作** | 單一功能可拆分為 ≥3 個獨立模組 | API + Service + Component + i18n |
| **多文件修改** | 涉及 >5 個文件且各自獨立 | 批量重構、模式統一 |
| **用戶明確要求** | 用戶提及「並行」「同時」「一起做」 | — |

### 標準執行流程

```
Step 1: 探索與規劃（主 Session）
  ├─ 讀取規劃文件（tech-spec、CHANGE doc、相關代碼）
  ├─ 識別可並行的獨立子任務
  ├─ 確認各子任務之間無文件衝突
  └─ 使用 TaskCreate 建立任務清單

Step 2: 並行派發（使用 Task tool）
  ├─ 為每個獨立子任務啟動 Agent（run_in_background: true）
  ├─ 每個 Agent 的 prompt 必須包含：
  │   ✅ 明確的實作範圍和預期產出
  │   ✅ 相關文件路徑
  │   ✅ 必須遵守的代碼規範引用
  │   ✅ 不可修改的文件（避免衝突）
  └─ 輸出啟動狀態表

Step 3: 監控與彙總（主 Session）
  ├─ 追蹤每個 Agent 的完成狀態
  ├─ Agent 完成後 TaskUpdate 更新狀態
  ├─ 如有 Agent 失敗，分析原因並決定重試或手動修復
  └─ 所有 Agent 完成後輸出最終狀態表

Step 4: 統一驗證（主 Session）
  ├─ npm run type-check（TypeScript 類型檢查）
  ├─ npm run lint（ESLint）
  ├─ npm run i18n:check（i18n 同步，如涉及 UI 文字）
  └─ 確認模組間整合正確性

Step 5: 統一 Commit（僅在用戶要求時）
```

### Agent 選擇指南

| 任務類型 | subagent_type | 用途 |
|----------|---------------|------|
| 功能實作（API + Service + Component） | `code-implementer` | 寫代碼，遵循規範 |
| 代碼品質檢查 | `code-quality-checker` | 檢查 9 條規則合規性 |
| i18n 翻譯同步 | `i18n-guardian` | 驗證 en/zh-TW/zh-CN 同步 |
| 架構設計驗證 | `architecture-reviewer` | 驗證設計決策 |
| 代碼探索與研究 | `Explore` | 快速搜尋代碼結構 |

### 進度追蹤格式

```markdown
| Agent | 任務 | 狀態 | 備註 |
|-------|------|------|------|
| task-1 | CHANGE-XXX: 描述 | ✅ 完成 / 🔄 進行中 / ❌ 失敗 | 修改 N 文件 |
| task-2 | CHANGE-YYY: 描述 | 🔄 進行中 | — |
```

### 限制與安全規則

- **禁止並行修改同一文件**: 如多個 Agent 需修改同一文件，必須序列化處理
- **i18n 統一處理**: 翻譯文件（`messages/`）的修改應在所有功能 Agent 完成後統一處理
- **Agent 內禁止 Commit**: 所有 git 操作由主 Session 統一執行
- **有依賴的任務不可並行**: 使用 TaskUpdate 的 `addBlockedBy` 管理依賴順序
- **衝突時使用 worktree**: 若無法避免文件衝突，使用 `isolation: "worktree"` 給 Agent 獨立副本

---

## 🌐 i18n 同步規則（🔴 必須遵守）

> **常見問題**：開發者在 TypeScript 中新增常量但忘記更新 i18n 翻譯，導致 `IntlError: MISSING_MESSAGE` 錯誤。

### i18n 命名空間完整列表（30 個/語言）

```
common, navigation, dialogs, auth, validation, errors, dashboard, global,
escalation, review, documents, rules, companies, reports, admin, confidence,
historicalData, termAnalysis, documentPreview, fieldMappingConfig,
promptConfig, dataTemplates, formats, templateFieldMapping, templateInstance,
templateMatchingTest, standardFields, referenceNumber, exchangeRate, region
```

> **重要**: 新增命名空間時，除了建立 3 個語言的 JSON 文件外，還必須在 `src/i18n/request.ts` 的 `namespaces` 陣列中加入新命名空間名稱。

### 核心同步規則

**當修改以下文件時，必須同步檢查 i18n：**
- `src/types/*.ts` 中含有 `label`、`description` 的 `export const` 常量
- `src/constants/*.ts` 中的任何常量
- 任何會在 UI 顯示的 enum/object

**新增功能模組時，必須同時：**
1. 建立 `messages/en/<namespace>.json`
2. 建立 `messages/zh-TW/<namespace>.json`
3. 建立 `messages/zh-CN/<namespace>.json`
4. 在 `src/i18n/request.ts` 的 `namespaces` 陣列中加入新命名空間

**開發完成前必須執行：** `npm run i18n:check`

> 📋 完整 i18n 規範、常量映射表與檢查清單：`.claude/rules/i18n.md`

---

## 🚨 技術障礙處理（🔴 必須遵守）

> **核心原則**: 遇到技術障礙時，**絕不擅自改變設計決策**。任何偏離原設計的方案必須獲得用戶確認。

| 禁止            | 說明                            |
| --------------- | ------------------------------- |
| ❌ 擅自替換組件 | 安裝失敗不能直接換其他組件      |
| ❌ 簡化功能     | 遇到困難不能減少功能            |
| ❌ 隱藏問題     | 不能不告知用戶就使用 workaround |

> 📋 完整處理流程、詢問模板與技術債務記錄格式：`.claude/rules/technical-obstacles.md`

---

## 開發後文檔更新檢查

每次完成開發任務後，依序檢查：
1. **是否完成 CHANGE/FIX？（🔴 必須）** → 更新規劃文件狀態為 ✅ 已完成
2. 是否涉及技術棧/架構變更？ → 更新 CLAUDE.md
3. 是否新增/修改模組的公開 API？ → 更新對應 index.ts
4. 是否發現新的開發規範/踩坑經驗？ → 更新 `.claude/rules/*.md`
5. 是否完成 Epic/Story？ → 更新進度追蹤

> 📋 完整檢查清單、自動提醒規則與更新範例：`claudedocs/reference/dev-checklists.md`

---

## 項目進度追蹤

全部 **22 個 Epic**（157+ Stories）已完成。項目已進入 **Phase 2 功能變更模式**，累計 33 CHANGE + 35 FIX。

> **進度唯一真實來源**: `docs/04-implementation/sprint-status.yaml`
> 📋 完整 Epic 進度表與變更記錄：`claudedocs/reference/project-progress.md`

---

## 🖥️ 跨電腦開發協作（多機同步）

本項目在多台電腦上同時開發，透過 GitHub 同步代碼。

### Git Slash Commands

| 命令 | 用途 | 說明 |
|------|------|------|
| `/git-status` | 快速狀態檢查 | 唯讀操作 — 一次輸出 Git 分支、同步狀態、Docker 服務、開發伺服器狀態 |
| `/git-sync` | 完整同步流程 | 拉取/推送 + 自動偵測依賴、Schema 變更並建議更新環境 |

### 切換電腦時的標準流程

```
1. /git-sync                          → 拉取最新代碼 + 同步環境
2. docker-compose up -d               → 確認 Docker 服務運行
3. npm run dev -- -p 3300             → 啟動開發伺服器
```

### 關鍵注意事項

- **DATABASE_URL 端口**: Docker PostgreSQL 對外映射端口為 **5433**（非 5432），`.env` 必須對應
- **`.next` 快取**: 跨電腦複製項目後必須 `rm -rf .next`，否則會有舊路徑導致 500 錯誤
- **Schema 同步**: 本項目使用 `prisma db push` 同步 Schema（非 migration），拉取後可能需要 `npx prisma db push --accept-data-loss`
- **種子資料**: 新環境首次設定需要 `npx prisma db seed` 建立基礎資料（角色、區域、城市、公司等）

> 📋 完整初始化指南：`docs/06-deployment/project-initialization-guide.md`

---

## 按需查閱文檔索引

| 需要了解 | 查閱路徑 |
|----------|----------|
| 完整目錄結構與 API 路由 | `claudedocs/reference/directory-structure.md` |
| 開發後檢查清單與提醒規則 | `claudedocs/reference/dev-checklists.md` |
| 項目進度表與版本記錄 | `claudedocs/reference/project-progress.md` |
| ClaudeDocs 完整索引 | `claudedocs/CLAUDE.md` |
| 服務啟動與問題排解 | `.claude/CLAUDE.md` |
| Sprint 狀態（唯一真實來源） | `docs/04-implementation/sprint-status.yaml` |
| JSDoc 模板與代碼範例 | `.claude/rules/general.md` |
| TypeScript 完整規範 | `.claude/rules/typescript.md` |
| 服務層規範 | `.claude/rules/services.md` |
| API 設計規範 | `.claude/rules/api-design.md` |
| 組件開發規範 | `.claude/rules/components.md` |
| 資料庫規範 | `.claude/rules/database.md` |
| 測試規範 | `.claude/rules/testing.md` |
| i18n 完整規範 | `.claude/rules/i18n.md` |
| 技術障礙處理流程 | `.claude/rules/technical-obstacles.md` |
| 並行 Agent 編排協議 | `CLAUDE.md` §開發編排協議 |
| 跨電腦開發與 Git 同步 | `CLAUDE.md` §跨電腦開發協作 |
| 項目初始化指南 | `docs/06-deployment/project-initialization-guide.md` |
| Git Slash Commands | `.claude/skills/git-sync/` + `.claude/skills/git-status/` |
| PRD | `docs/01-planning/prd/prd.md` |
| 系統架構設計 | `docs/02-architecture/` |
| Tech Specs | `docs/03-stories/tech-specs/` |
| 實施上下文 | `docs/04-implementation/implementation-context.md` |

---

## 版本資訊

- **CLAUDE.md 版本**: 3.2.0
- **最後更新**: 2026-03-14
- **變更記錄**: 新增跨電腦開發協作章節、Git Slash Commands、初始化指南引用（v3.1.0 → v3.2.0）
- **完整版本歷史**: `claudedocs/reference/project-progress.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laitim2001)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laitim2001)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
