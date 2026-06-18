## comfortplus-erp-new

> 護理用品（成人紙尿布）ERP 系統，服務 B2B 機構客戶（護理之家、長照中心）+ B2C 電商通路。

# ComfortPlus ERP

護理用品（成人紙尿布）ERP 系統，服務 B2B 機構客戶（護理之家、長照中心）+ B2C 電商通路。

## Tech Stack

- **Framework**: Next.js 16 (App Router) + React 19 + TypeScript
- **DB**: PostgreSQL 16 (Docker, port 5434) + Prisma 7 (modular schema in `prisma/schema/`)
- **Auth**: NextAuth 5 (JWT, 8hr session)
- **UI**: shadcn/ui (Base UI) + Tailwind CSS 4 + Recharts
- **AI**: Ollama (local GPU) / Anthropic Claude (cloud) — dual provider in `src/lib/ai.ts`
- **i18n**: zh-TW (default) / en / th — dictionary in `src/lib/i18n/locales/`

## 開發指令

```bash
npm run dev          # 開發模式 (port 3001)
npm run build        # 正式打包
npx tsc --noEmit     # 型別檢查（必須零錯誤）
npx tsx prisma/seed.ts  # 建立測試資料
npx prisma db push   # 同步 schema 到 DB
```

## 專案結構

```
prisma/schema/         # 模組化 Prisma schema（01-enums ~ 19-calendar）
src/app/(dashboard)/   # 所有 Dashboard 頁面
src/app/api/           # API routes（147+ endpoints）
src/components/
  ├── dashboard/       # 6 個角色專屬 Dashboard 元件
  ├── ai/              # AI 助手浮動視窗
  ├── ui/              # shadcn/ui 基礎元件
  └── layout/          # Sidebar, Header, MobileNav
src/lib/
  ├── ai.ts            # AI provider 抽象層（Ollama/Anthropic）
  ├── ai-skills.ts     # AI 可執行技能（報價/出貨/庫存/KPI）
  ├── api-error.ts     # 統一 API 錯誤處理
  ├── audit.ts         # 稽核日誌
  ├── kpi-check.ts     # KPI 里程碑自動通知
  ├── logger.ts        # 結構化日誌
  ├── notify.ts        # 多通路通知（系統/LINE/Email）
  ├── rate-limit.ts    # API Rate limiter
  ├── scope.ts         # 角色資料範圍過濾
  └── sequence.ts      # 單號產生器
```

## 角色系統

11 個角色，每個角色有專屬 Dashboard：
- SUPER_ADMIN / GM → 經營全局儀表板
- SALES_MANAGER → 團隊管理（含 KPI）
- SALES / CS / CARE_SUPERVISOR → 個人工作台
- WAREHOUSE_MANAGER / WAREHOUSE → 倉儲作業台
- FINANCE → 財務工作台
- PROCUREMENT → 採購工作台
- ECOMMERCE → GM 視角（Phase 2 獨立）

## 關鍵設計決策

- **資料範圍過濾**：SALES 只看自己的訂單/客戶/報價，SALES_MANAGER 看全部。邏輯在 `src/lib/scope.ts`
- **庫存並發**：訂單確認時用 `SELECT FOR UPDATE` 行級鎖防止超賣
- **KPI 自動連動**：建單後自動觸發 `checkKpiMilestone()`，達 50/80/100% 通知
- **分頁格式**：所有 list API 回傳 `{ data: T[], pagination: { page, pageSize, total, totalPages } }`
- **觸控優化**：手機上 button/input 最小 44px，使用 `active:scale-[0.97]` 按壓回饋
- **i18n**：UI 文字用 `dict.xxx` 取值，AI Skill 關鍵字保持中文（用於意圖辨識）
- **三套入庫系統**（5-7 已知技術債）：`InboundRecord`（海運到倉）/ `WmsInbound`（WMS 作業）/ `ProductionReceipt`（製令入庫）三套並存，各有獨立庫存觸發點。長期目標：合併為 `InboundRecord + sourceType` 欄位，需 schema migration。
- **兩套 QC 系統**（5-8 已知技術債）：`/api/inbound/[id]/qc` 處理進貨驗收，`QualityCheck` model + `/api/qc` 處理製令/獨立品檢。兩套已在 Phase 3 串接（QC pass → InventoryLot，fail → DefectiveGoods）。長期目標：統一為 QualityCheck + sourceType。
- **訂單 vs 銷貨單職責**（5-9 設計決策）：`SalesOrder` = 商務合約（確認客戶、金額、條件）；`SalesInvoice` = 出庫憑證（實際出倉記錄，可多次分批）。訂單 CONFIRMED 自動建銷貨單，出貨扣庫存。

## 測試帳號

| 帳號 | 密碼 | 角色 |
|------|------|------|
| admin@comfortplus.com | admin1234 | SUPER_ADMIN |
| gm@comfortplus.com | gm12345678 | GM |
| manager@comfortplus.com | manager1234 | SALES_MANAGER |
| sales@comfortplus.com | sales1234 | SALES |
| warehouse@comfortplus.com | warehouse1234 | WAREHOUSE |
| wm@comfortplus.com | wm12345678 | WAREHOUSE_MANAGER |
| finance@comfortplus.com | finance1234 | FINANCE |
| procurement@comfortplus.com | procurement1234 | PROCUREMENT |

## 注意事項

- 不要修改 `prisma/schema.prisma`（舊版單檔），實際 schema 在 `prisma/schema/` 目錄
- API 錯誤處理用 `handleApiError(error, 'module.action')` — 生產環境隱藏 stack trace
- 所有 API route 必須有 `auth()` 檢查（除了 `/api/health` 和 `/api/auth/`）
- `/api/cron` 用 `CRON_SECRET` Bearer token 認證，不走 session
- 上傳檔案上限 10MB，只接受 JPEG/PNG/WebP/HEIC

---
> Source: [ah3-max/comfortplus-erp-new](https://github.com/ah3-max/comfortplus-erp-new) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
