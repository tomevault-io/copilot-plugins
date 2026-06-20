## 2026-ai-adv-homework-course01-v1

> 本檔案為 Claude Code 與本專案協作時的入口指南。詳細內容請參考 `docs/` 目錄。

# CLAUDE.md

本檔案為 Claude Code 與本專案協作時的入口指南。詳細內容請參考 `docs/` 目錄。

## 專案概述

**花漾生活 Flower Life** — 一個花卉電商網站 Demo，使用 Node.js + Express 4 + better-sqlite3 構建後端 REST API，搭配 EJS 伺服器端模板 + Vue 3 (CDN) + Tailwind CSS 4 構建前後台頁面。功能涵蓋：

- 使用者註冊／登入（JWT 認證，bcrypt 雜湊）
- 商品瀏覽（前台分頁列表、詳情）
- 購物車（雙模式認證：JWT 會員 或 X-Session-Id 訪客）
- 訂單建立（從購物車結帳，含庫存扣減 transaction）、模擬付款、訂單查詢
- 後台管理（商品 CRUD、訂單列表／詳情／狀態篩選）
- OpenAPI 3.0 規格自動產生（透過 swagger-jsdoc 從 JSDoc 註解）

## 常用指令

從 `package.json` 的 `scripts` 偵測：

| 指令 | 用途 |
|------|------|
| `npm start` | 建置 Tailwind CSS（壓縮）並啟動伺服器 |
| `npm run dev:server` | 僅啟動伺服器（不重新建置 CSS） |
| `npm run dev:css` | Tailwind CSS watch 模式（開發時與 dev:server 並行執行） |
| `npm run css:build` | 一次性建置 Tailwind CSS（壓縮版） |
| `npm run openapi` | 從路由 JSDoc 註解產生 `openapi.json` |
| `npm test` | 執行 vitest 測試（單次執行，非 watch 模式） |

預設 port：`3001`（可由環境變數 `PORT` 覆寫）。

## 關鍵規則

- **統一 API 回應格式**：所有 `/api/*` 端點必須回傳 `{ data, error, message }` 三鍵物件；成功時 `error: null`，失敗時 `data: null` 且 `error` 為錯誤代碼字串（例如 `VALIDATION_ERROR`、`UNAUTHORIZED`、`NOT_FOUND`、`CONFLICT`、`STOCK_INSUFFICIENT`、`CART_EMPTY`、`INVALID_STATUS`、`INTERNAL_ERROR`）。
- **JWT_SECRET 為啟動必要條件**：`server.js` 在啟動時若偵測不到 `process.env.JWT_SECRET` 會 `process.exit(1)`。本機開發請複製 `.env.example` 為 `.env` 並填入有效值。
- **資料庫初始化內建於應用程式**：`require('./src/database')` 會自動建立資料表與 seed 資料（admin 帳號 + 8 筆商品）；首次啟動會在專案根目錄建立 `database.sqlite`，並啟用 WAL 模式與 foreign keys。
- **購物車雙模式認證**：`/api/cart/*` 端點同時接受 `Authorization: Bearer <JWT>` 或 `X-Session-Id: <uuid>` 請求標頭，二擇一即可，且 cart_items 表會根據來源寫入 `user_id` 或 `session_id`。
- **JSDoc 即 OpenAPI 規格**：在新增／修改路由時必須同步維護 `@openapi` JSDoc 註解；前端 / 第三方串接的 API 文件由 `npm run openapi` 從這些註解產生。
- **功能開發使用 `docs/plans/` 記錄計畫；完成後移至 `docs/plans/archive/`**（命名格式 `YYYY-MM-DD-<feature-name>.md`，詳見 `docs/DEVELOPMENT.md`）。

## 詳細文件

- [./docs/README.md](./docs/README.md) — 項目介紹、技術棧、快速開始
- [./docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) — 架構、目錄結構、啟動流程、API 路由表、認證機制、資料庫 schema
- [./docs/DEVELOPMENT.md](./docs/DEVELOPMENT.md) — 開發規範、命名規則、新增 API/middleware/DB 的步驟、計畫歸檔流程、環境變數表
- [./docs/FEATURES.md](./docs/FEATURES.md) — 功能列表、行為描述、查詢參數、業務邏輯、錯誤碼
- [./docs/TESTING.md](./docs/TESTING.md) — 測試規範、執行順序、撰寫指南、常見陷阱
- [./docs/CHANGELOG.md](./docs/CHANGELOG.md) — 更新日誌

---
> Source: [Holly05590/2026-ai-adv-homework-course01-v1](https://github.com/Holly05590/2026-ai-adv-homework-course01-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
