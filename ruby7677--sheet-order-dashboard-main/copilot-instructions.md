## sheet-order-dashboard-main

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

這是一個蘿蔔糕訂購系統的後台管理 Dashboard，具備訂單管理、客戶管理、統計分析等功能。系統採用前後端分離架構，前端使用 React + TypeScript + Tailwind CSS，後端包含 PHP API（傳統）和 Cloudflare Workers API（現代化）兩套方案，並整合 Supabase 作為新的資料存儲解決方案。

## 架構設計

### 前端架構
- **技術棧**: React 18 + TypeScript + Vite + Tailwind CSS v4 + shadcn/ui
- **狀態管理**: TanStack Query 用於服務器狀態管理，React Context 用於認證狀態  
- **路由**: React Router v6，主路由在 `src/App.tsx`，管理路由在 `src/routes/adminRoutes.tsx`
- **UI 組件**: shadcn/ui 組件庫，完整的 Radix UI 系統
- **圖表**: Recharts 用於數據可視化
- **錯誤處理**: ErrorBoundary 組件包裝整個應用

### 後端架構
- **傳統 API**: PHP API (`api/` 目錄) - Google Sheets 整合，作為後備方案
- **現代 API**: Cloudflare Workers (`sheet-order-api/` 目錄) - 使用 Hono 框架 + OpenAPI
- **新資料層**: Supabase 整合 (`src/integrations/supabase/`)
- **API 路由**: 支援現代化 REST 端點和 PHP 兼容端點

### 動態 API 配置架構
系統實現智能的 API 端點切換機制：
1. **生產環境**: 優先使用 Cloudflare Workers API (`https://sheet-order-api.ruby7677.workers.dev`)
2. **本地開發**: 嘗試本地 Workers API (`http://127.0.0.1:5714`)，失敗則自動降級到 PHP API
3. **環境檢測**: 基於 hostname 和 port 自動判斷運行環境
4. **降級機制**: 多層降級確保服務可用性

### 數據來源與整合
1. **Google Sheets**: 主要數據來源（訂單表、客戶名單表）
2. **Supabase**: 現代化資料庫存儲
3. **Cloudflare KV**: Workers API 的快取層
4. **前端快取**: TanStack Query 管理 API 快取

### 電話號碼處理架構
- **標準化**: 移除所有非數字字符 (`/[^0-9]/g`)
- **比對邏輯**: 使用後九碼進行匹配，與 PHP API 保持一致
- **快取策略**: 以標準化後的電話號碼作為快取鍵

## 常用指令

### 開發環境
```powershell
# 啟動前端開發服務器 (端口 8080)
npm run dev

# 啟動前端開發服務器 (除錯模式)
npm run dev:debug

# 清除快取並啟動 (Windows PowerShell)
npm run dev:clean
```

### 建置和部署
```powershell
# 生產環境建置
npm run build

# 開發環境建置
npm run build:dev

# 預覽建置結果
npm run preview

# 清除 node_modules 快取和 dist (Windows PowerShell)
npm run clean
```

### 程式碼品質
```powershell
# ESLint 檢查和修復
npm run lint
```

### Cloudflare Workers API
```powershell
# 進入 Workers 目錄
cd sheet-order-api

# 本地開發 Workers (端口 5714，使用 wrangler dev)
npm run dev

# 部署到 Cloudflare (需要 Cloudflare 帳戶和權限)
npm run deploy

# 生成 TypeScript 類型
npm run cf-typegen
```

### Supabase 開發
```bash
# 啟動本地 Supabase (如果有設定)
supabase start

# 推送資料庫 migration
supabase db push

# 生成 TypeScript 類型
supabase gen types typescript --local > src/integrations/supabase/types.ts
```

## 核心服務架構

### API 服務層 (`src/services/`)
- **orderService.ts**: 訂單管理服務，包含動態 API 配置和降級邏輯
- **customerService.ts**: 客戶管理服務，電話號碼標準化和客戶訂單歷史
- **migrationService.ts**: 數據遷移服務
- **secureApiService.ts**: 安全 API 調用服務

### 組件架構 (`src/components/`)
- **主頁面**: `src/pages/Index.tsx` - 雙模式界面（訂單/客戶管理）
- **側邊欄**: `ModernSidebar.tsx` - 模式切換和統計顯示
- **控制面板**: `CompactControlPanel.tsx` - 統合篩選器和操作按鈕
- **詳情組件**: `OrderDetail.tsx`, `CustomerDetail.tsx` - 詳細信息展示
- **快取清除**: 所有服務都提供 `clearCache()` 函數

### Workers API 架構 (`sheet-order-api/src/`)
- **框架**: Hono + chanfana (OpenAPI)
- **端點**: RESTful API + PHP 兼容端點
- **快取**: Cloudflare KV 快取服務
- **環境**: 生產和開發環境分離配置

## 環境配置

### 開發端口
- 前端開發服務器: `8080` (已配置在 package.json 和 vite.config.ts)
- Cloudflare Workers 本地: `5714` (已配置在 sheet-order-api/wrangler.jsonc)
- 測試替代端口: `8082` 或 `5714`

### 關鍵 API 端點
- **訂單**: `/api/orders` (Workers) 或 `/api/get_orders_from_sheet.php` (PHP)
- **客戶**: `/api/customers` (Workers) 或 `/api/get_customers_from_sheet.php` (PHP)  
- **客戶訂單**: `/api/customers/orders` (Workers) 或 `/api/get_customer_orders.php` (PHP)
- **訂單更新**: `/api/orders/status`, `/api/orders/payment`, `/api/orders/items`

### 跨域和代理配置
- 開發環境配置了詳細的 CORS 和代理設定 (`vite.config.ts`)
- 支援多域名和 iframe 嵌入
- URI 解碼錯誤處理和安全路徑重寫

## 開發注意事項

### 程式碼風格
- 檔案大小限制: 單檔嚴格控制在 500 行以內
- TypeScript 嚴格模式啟用
- 主動將過大檔案拆分為邏輯清晰的小模組

### API 開發策略
- **優先順序**: Cloudflare Workers API > PHP API
- **降級機制**: 自動檢測和降級，確保服務穩定性
- **統一管理**: 所有 API 調用透過對應的 service 檔案管理
- **快取策略**: 前端 TanStack Query + Workers KV 雙層快取

### 客戶訂單歷史
- **數據來源**: Google Sheets「客戶名單」工作表（非訂單表）
- **電話比對**: 使用後九碼標準化比對邏輯
- **API 端點**: 優先使用 `/api/customers/orders`，降級到 `/api/get_customer_orders.php`
- **快取鍵**: 使用標準化電話號碼的後九碼

### 環境變數管理
- **Vite**: 環境變數前綴 `VITE_`
- **Workers**: 環境變數在 `wrangler.jsonc` 中配置，支援生產/開發環境
- **Supabase**: 配置透過 `src/integrations/supabase/client.ts`

### iframe 模式支援
- 主頁面支援 iframe 嵌入模式
- 透過 PostMessage API 進行父子窗口通信
- 簡化界面在 iframe 模式下隱藏不必要的導航元素

### 雙模式界面
- **訂單模式**: 訂單管理、篩選、統計、批量操作
- **客戶模式**: 客戶管理、統計、訂單歷史查詢
- 模式切換透過 `ModernSidebar` 組件控制

## TypeScript 配置

### 嚴格模式設定
- **TypeScript 配置**: 採用專案引用結構 (`tsconfig.json`, `tsconfig.app.json`, `tsconfig.node.json`)
- **嚴格檢查**: 啟用 `noImplicitAny`, `noUnusedParameters`, `noUnusedLocals`, `strictNullChecks`
- **路徑別名**: `@/*` 指向 `./src/*`
- **編譯選項**: 允許 JavaScript 檔案，跳過 lib 檢查以提升編譯速度

### Workers TypeScript 配置
- **獨立配置**: `sheet-order-api/tsconfig.json` 使用 Cloudflare Workers 類型
- **類型生成**: 使用 `wrangler types` 自動生成 Cloudflare Workers 類型定義

## Windows 開發環境

### 指令相容性
- **終端機**: 使用 PowerShell 相容語法
- **指令分隔**: 使用分號 (`;`) 分隔多個指令
- **避免**: 不使用 `curl` 指令，改用 fetch API 或其他方法
- **路徑處理**: 支援 Windows 路徑格式，URI 解碼錯誤處理

### 代理和 CORS 設定
- **開發代理**: 自動將 `/api` 路徑重寫為 `/sheet-order-dashboard-main/api`
- **跨域設定**: 支援多個允許的域名和 iframe 嵌入
- **錯誤處理**: URI 解碼失敗時的安全後備機制
- **安全標頭**: CSP 和 CORS 標頭配置以支援開發和生產環境

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ruby7677) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
