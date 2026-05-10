## exhibition-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

這是一個展覽管理系統，使用 Next.js 15 (App Router) + React 19 + Prisma + PostgreSQL 構建的全端應用程式。系統包含展覽管理、作品管理、預約叫號、任務看板、前台模板編輯等功能模組。

## 開發指令

### 常用命令

```bash
# 開發環境
pnpm dev              # 啟動開發服務器 (http://localhost:3000)

# 資料庫操作
pnpm db:generate      # 生成 Prisma Client (修改 schema 後必須執行)
pnpm db:push          # 推送 Schema 到資料庫 (開發環境快速同步)
pnpm db:migrate       # 執行資料庫遷移 (生產環境用)
pnpm db:studio        # 開啟 Prisma Studio 視覺化介面
pnpm db:seed          # 填充測試資料

# 構建與程式碼檢查
pnpm build            # 構建生產版本
pnpm start            # 啟動生產服務器
pnpm lint             # ESLint 檢查

# 測試
pnpm test             # Vitest 監聽模式
pnpm test:run         # 執行單元測試 (一次性)
pnpm test:coverage    # 執行測試含覆蓋率報告
pnpm test:ui          # Vitest UI 介面
pnpm test:e2e         # Playwright E2E 測試
pnpm test:e2e:ui      # Playwright UI 模式
pnpm test:e2e:report  # 查看 E2E 測試報告
```

### 本地服務需求

開發前需確保以下服務在本地運行：
- **PostgreSQL 16**: 運行在 5432 端口
- **Redis 7**: 運行在 6379 端口
- **MinIO** (可選): 如需測試檔案儲存功能，運行在 9000 端口

## 專案架構

### 技術決策

- **前端框架**: Next.js 15 使用 App Router (不是 Pages Router)
- **UI 元件**: TailwindCSS + shadcn/ui (25 個元件，包含表單、資料展示、互動、通知等)
- **資料庫**: PostgreSQL 16 + Prisma ORM (單一 schema.prisma 包含 52 個表)
- **認證系統**: NextAuth.js v5 Beta + 自定義用戶管理
- **狀態管理**: Zustand (全局) + TanStack Query (服務端狀態)
- **套件管理**: 必須使用 pnpm (不要使用 npm 或 yarn)

### 目錄結構說明

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # 認證相關頁面 (登入/註冊)
│   ├── admin/             # 管理後台 (需登入)
│   │   ├── exhibitions/   # 展覽管理前端（列表、新建、編輯、詳情）
│   │   ├── users/         # 用戶管理前端（列表、新建、編輯、詳情）
│   │   ├── artworks/      # 作品管理前端（列表、新建、編輯、詳情）
│   │   ├── teams/         # 團隊管理前端（列表、新建、編輯、詳情）
│   │   ├── documents/     # 文件管理前端（列表、上傳、詳情）
│   │   ├── sponsors/      # 贊助商管理前端（列表、新建、編輯、詳情）
│   │   ├── venues/        # 場地管理前端（列表、新建、編輯、詳情）
│   │   ├── templates/     # 前台模板管理（列表、新建、編輯器）
│   │   ├── homepage-editor/ # 首頁編輯器（Wix-like 拖放編輯）
│   │   ├── settings/      # 系統設定頁面
│   │   └── page.tsx       # 後台首頁（儀表板）
│   ├── exhibitions/       # 前台展覽展示頁面
│   ├── api/               # API 路由
│   │   ├── auth/          # 認證 API
│   │   ├── exhibitions/   # 展覽 CRUD API
│   │   ├── teams/         # 團隊 CRUD API
│   │   ├── team-members/  # 團隊成員 CRUD API
│   │   ├── artworks/      # 作品 CRUD API
│   │   ├── users/         # 用戶 CRUD API
│   │   ├── sponsors/      # 贊助商 CRUD API
│   │   ├── venues/        # 場地 CRUD API
│   │   ├── documents/     # 文件 CRUD API
│   │   ├── site-templates/# 前台模板 CRUD API
│   │   ├── system-settings/# 系統設定 API
│   │   ├── upload/        # 檔案上傳 API
│   │   └── audit-logs/    # 審計日誌查詢 API
│   └── page.tsx           # 首頁
├── components/            # React 元件
│   ├── ui/                # shadcn/ui 基礎元件
│   ├── editor/            # 編輯器元件（Craft.js & Wix-like）
│   │   ├── components/    # 可編輯元件（Container, Text, Button, Image）
│   │   ├── WixLikeEditor/ # Wix-like 編輯器元件
│   │   └── settings/      # 元件屬性設定面板
│   ├── forms/             # 表單元件
│   │   ├── ExhibitionForm.tsx
│   │   ├── UserForm.tsx
│   │   ├── ArtworkForm.tsx
│   │   ├── TeamForm.tsx
│   │   ├── SponsorForm.tsx
│   │   ├── VenueForm.tsx
│   │   ├── DocumentUploadForm.tsx
│   │   └── TemplateCreateForm.tsx
│   └── layout/            # 佈局元件
│       ├── AdminSidebar.tsx
│       ├── AdminHeader.tsx
│       └── AdminLayoutClient.tsx
├── lib/                   # 核心工具
│   ├── auth.ts            # NextAuth 配置
│   ├── auth.config.ts     # NextAuth 詳細配置
│   ├── prisma.ts          # Prisma 客戶端單例
│   ├── redis.ts           # Redis 連接
│   ├── minio.ts           # MinIO 檔案儲存客戶端
│   ├── audit-log.ts       # 審計日誌工具
│   ├── api-response.ts    # 統一 API 響應格式
│   ├── api-client.ts      # 前端 API 客戶端
│   ├── scroll-animations.ts  # 滾動動畫工具 (6種效果)
│   ├── preset-templates.ts   # 預設模板配置
│   ├── utils.ts           # 通用工具函數
│   └── validations/       # Zod 驗證 schemas
│       ├── exhibition.ts
│       ├── artwork.ts
│       ├── team.ts
│       ├── sponsor.ts
│       ├── venue.ts
│       └── user.ts
├── styles/                # 全局樣式
│   └── scroll-animations.css  # 滾動動畫 CSS 定義
├── hooks/                 # Custom React Hooks
│   └── use-toast.ts       # Toast 通知 hook
├── types/                 # TypeScript 類型定義
│   └── next-auth.d.ts    # NextAuth 類型擴展
└── env.ts                 # 環境變數驗證
```

### 資料庫架構重點

1. **用戶系統**: 三種角色 (SUPER_ADMIN, CURATOR, TEAM_LEADER)
2. **展覽體系**: Exhibition → Team → TeamMember → Artwork
3. **預約系統**: ReservationSetting → ReservationSlot → Reservation
4. **任務看板**: Board → List → Card (支援拖放排序)
5. **檔案管理**: Document 表關聯多個實體，統一管理所有檔案
6. **前台模板**: SiteTemplate 儲存 Craft.js JSON，支援發布管理
7. **系統設定**: SystemSetting 鍵值對儲存，支援 JSON 資料

## 開發注意事項

### 必須遵循的規則

1. **資料庫操作**:
   - 修改 schema.prisma 後必須執行 `pnpm db:generate`
   - 絕不使用 `--accept-data-loss` 參數
   - 使用事務處理批量操作

2. **API 設計**:
   - 所有 API 路由放在 `src/app/api/` 目錄
   - 使用 Zod 驗證請求數據
   - 返回統一的錯誤格式

3. **認證授權**:
   - 使用 NextAuth 的 `auth()` 函數檢查登入狀態
   - 管理後台路由必須檢查用戶角色
   - API 路由需加入認證中間件

4. **前端開發**:
   - 使用 Server Components 優先
   - Client Components 加上 'use client' 指令
   - 表單使用 Server Actions 處理

5. **UI 元件規範**:
   - **後台管理頁面必須使用 shadcn/ui 元件**，不要使用原生 HTML 元素
   - 表單元件優先順序：Label + Input/Select/Textarea/Switch > 原生 HTML
   - 按鈕統一使用 Button 元件（variant="outline" 用於次要操作）
   - 使用 space-y-2 保持元件間距一致
   - 確保 Label 和 Input 使用 htmlFor/id 配對（無障礙設計）

### 現有功能模組

**Phase 1: 核心基礎建設** ✅ 已完成
- **認證系統**: NextAuth v5 + JWT strategy（已修正配置）
- **環境變數驗證**: @t3-oss/env-nextjs 確保配置安全
- **UI 元件庫**: shadcn/ui（25 個元件）
- **管理後台**: 增強版儀表板，7 個統計卡片 + 快速操作選單
- **展覽管理 API**: 完整 CRUD + 分頁查詢 + 審計日誌
- **作品管理 API**: 完整 CRUD + 細粒度權限控制
- **檔案儲存系統**: MinIO 整合（支援圖片/影片/文件上傳）
- **審計日誌**: 自動記錄所有重要操作（用戶註冊、展覽/作品 CRUD 等）
- **滾動動畫庫**: 6 種精選 Scroll-Driven Animations
- **錯誤處理**: 統一的 API 響應格式 + Toast 通知

**Phase 2: 展覽與內容管理** ✅ 已完成 (100%)
- **完整前後端實現**: 展覽、用戶、作品、團隊、文件、贊助商、場地管理
- **文件管理系統**: 支援 PDF 線上預覽，其他格式下載
- **贊助商管理**: Logo 展示、贊助等級、聯絡資訊管理
- **場地管理**: 平面圖展示、容納人數追蹤

**Phase 3: 前台模板系統** ✅ 已完成 (100%)
- **Craft.js 編輯器**: 拖放式頁面編輯器，支援 4 種基礎元件
- **預設模板系統**: 4 種專業預設模板（空白、極簡、現代、藝廊）
- **模板管理**: 創建、編輯、預覽、發布流程完整
- **前台渲染**: 支援自訂模板和預設頁面雙模式

**Phase 4: Wix-like 首頁編輯器** ✅ 已完成
- **拖放編輯器**: 類似 Wix 的視覺化編輯體驗
- **多種區塊**: Hero、Features、Gallery、CTA、Footer 等
- **即時預覽**: 編輯即見所得
- **系統設定 API**: 儲存首頁配置到 SystemSetting 表

### API 使用範例

**展覽管理**
```typescript
import { apiClient } from '@/lib/api-client'

// 獲取展覽列表
const { exhibitions, pagination } = await apiClient.get('/exhibitions?page=1&limit=10')

// 創建展覽（自動記錄審計日誌）
const exhibition = await apiClient.post('/exhibitions', {
  name: '2024 畢業展',
  year: 2024,
  slug: '2024-graduation',
  startDate: '2024-06-01',
  endDate: '2024-06-30'
})

// 更新展覽
const updated = await apiClient.patch(`/exhibitions/${id}`, {
  status: 'PUBLISHED'
})
```

**檔案上傳**
```typescript
// 上傳圖片
const formData = new FormData()
formData.append('file', file)
formData.append('type', 'image')
formData.append('relatedId', exhibitionId)
formData.append('relatedType', 'exhibition')

const result = await fetch('/api/upload', {
  method: 'POST',
  body: formData
})

// 獲取預簽名下載 URL
const { downloadUrl } = await apiClient.get(`/upload?fileName=${fileName}`)
```

**審計日誌**
```typescript
import { createAuditLog, AuditAction } from '@/lib/audit-log'

// 記錄自定義操作
await createAuditLog({
  action: AuditAction.EXHIBITION_PUBLISH,
  userId: user.id,
  entityType: 'Exhibition',
  entityId: exhibition.id,
  metadata: {
    publishedAt: new Date(),
    status: 'PUBLISHED'
  },
  ipAddress: '1.2.3.4',
  userAgent: 'Mozilla/5.0...'
})

// 查詢審計日誌（僅 SUPER_ADMIN）
const { logs, pagination } = await apiClient.get('/audit-logs?page=1&limit=50')
```

**滾動動畫（前台專用）**
```typescript
import { scrollAnimate, scrollAnimateList, EXHIBITION_PRESETS } from '@/lib/scroll-animations'
import '@/styles/scroll-animations.css'

// 基本使用：單一元素
<div {...scrollAnimate('fade-in-up')}>
  作品卡片內容
</div>

// 帶延遲：錯開動畫時間
<div {...scrollAnimate('scale-reveal', 2)}>
  第二個元素延遲 0.2 秒
</div>

// 批量使用：自動為列表加上遞增延遲
{scrollAnimateList(artworks, 'fade-in-up').map(({ props, item }) => (
  <div key={item.id} {...props}>
    <img src={item.imageUrl} alt={item.title} />
    <h3>{item.title}</h3>
  </div>
))}

// 使用預設配置
<section {...scrollAnimate(EXHIBITION_PRESETS.exhibitionHero)}>
  展覽封面區塊
</section>

// 直接使用 data 屬性
<h1 data-animate="blur-in">展覽標題</h1>
<p data-animate="accordion" data-animate-delay="1">說明文字</p>
```

**瀏覽器支援**: Chrome/Edge 115+ 原生支援，Safari 需 polyfill，不支援時自動降級

**shadcn/ui 元件使用範例（後台管理專用）**
```typescript
// 表單基本結構
<div className="space-y-2">
  <Label htmlFor="name">
    欄位名稱 <span className="text-red-500">*</span>
  </Label>
  <Input
    id="name"
    type="text"
    name="name"
    value={formData.name}
    onChange={handleChange}
    required
    placeholder="請輸入內容"
  />
  <p className="text-sm text-muted-foreground">輔助說明</p>
</div>

// Select 元件
<Select
  value={formData.status}
  onValueChange={(value) => setFormData((prev) => ({ ...prev, status: value }))}
>
  <SelectTrigger id="status">
    <SelectValue placeholder="選擇狀態" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="DRAFT">草稿</SelectItem>
    <SelectItem value="PUBLISHED">已發布</SelectItem>
  </SelectContent>
</Select>

// Switch 元件
<div className="flex items-center space-x-2">
  <Switch
    id="isActive"
    checked={formData.isActive}
    onCheckedChange={(checked) => setFormData(prev => ({ ...prev, isActive: checked }))}
  />
  <Label htmlFor="isActive" className="cursor-pointer">啟用功能</Label>
</div>

// 按鈕組
<div className="flex justify-end space-x-4">
  <Button type="button" variant="outline" onClick={() => router.back()}>
    取消
  </Button>
  <Button type="submit" disabled={loading}>
    {loading ? '處理中...' : '確認送出'}
  </Button>
</div>
```

### 待實作功能

- 預約叫號系統（ReservationSetting, ReservationSlot, Reservation 表已建立）
- 任務看板（Board, List, Card 表已建立，Trello-like 功能）
- 即時通訊（Socket.io 整合）
- Email 系統（SMTP 配置已準備）
- 數據匯出功能
- 全站 RWD 優化
- 進階搜尋功能

## 錯誤處理

- 所有錯誤必須完整顯示在前端
- API 錯誤使用統一格式: `{ success: false, error: string }`
- 使用 try-catch 包裹資料庫操作
- 記錄錯誤到審計日誌 (audit_logs 表)

## 效能優化建議

1. 使用 Redis 快取頻繁查詢的數據
2. 圖片使用 Next.js Image 元件優化
3. 大量數據使用分頁查詢
4. 使用 Prisma 的 select 減少數據傳輸

## 測試環境

- 第一個註冊的用戶自動成為 SUPER_ADMIN
- Prisma Studio: 執行 `pnpm db:studio` 可視覺化管理資料庫
- MinIO 控制台 (http://localhost:9001) - 如有安裝

---
> Source: [Eric920418/exhibition-system](https://github.com/Eric920418/exhibition-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
