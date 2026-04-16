## sfda-nexus

> - **使用 Docker MySQL 容器**，不要啟動本機 MySQL 服務

# SFDA Nexus 項目規則

## 資料庫配置規則

### MySQL 服務
- **使用 Docker MySQL 容器**，不要啟動本機 MySQL 服務
- 容器名稱：`mysql-server`
- 連接方式：`docker exec -i mysql-server mysql -u root -pMyPwd@1234 sfda_nexus`
- 執行 SQL 腳本：`docker exec -i mysql-server mysql -u root -pMyPwd@1234 sfda_nexus < script.sql`
- 如果需要停止本機 MySQL：`brew services stop mysql`

### MySQL 中文亂碼解決方案
- **常見問題**：插入中文數據後出現亂碼，通常是字符集配置問題
- **標準解決方案**：
  1. 執行字符集修復腳本：`docker exec -i mysql-server mysql -u root -pMyPwd@1234 --default-character-set=utf8mb4 < backend/database/scripts/fix_mysql_charset.sql`
  2. 腳本會自動設置：
     - `SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci`
     - `ALTER DATABASE sfda_nexus CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci`
     - `ALTER TABLE 表名 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci`
  3. 驗證字符集：檢查 `@@character_set_client`, `@@character_set_connection`, `@@character_set_results` 都應該是 `utf8mb4`
- **預防措施**：執行任何包含中文的 SQL 腳本時，都應該使用 `--default-character-set=utf8mb4` 參數

### 資料庫腳本管理
- 所有資料庫相關腳本統一放在 `backend/database/scripts/` 目錄
- 腳本命名規範：
  - 測試腳本：`test_功能名稱.js`
  - 種子腳本：`seed_數據類型.js`
  - 維護腳本：`功能描述.js`
  - SQL 腳本：`add_表名.sql` 或 `update_功能.sql`
  - 修復腳本：`fix_問題描述.sql`

## 開發環境配置

### 後端服務
- 開發模式：`NODE_ENV=development`
- 啟動命令：`cd backend && npm start`
- 調試模式：已啟用詳細的 SQL 查詢日誌

### 前端服務
- 開發服務器：`cd frontend && npm run dev`
- 端口：5173（Vite 默認）

## 調試規則

### SQL 調試
- 開發模式下會自動打印完整的 SQL 查詢和參數
- 錯誤信息包含：SQL 語句、參數、錯誤代碼、格式化 SQL
- 使用 `console.error` 直接輸出到控制台，便於調試

### API 調試
- 前端 API 層統一使用 `api/` 目錄下的模塊
- 所有 HTTP 請求通過配置好的 axios 實例
- 自動處理認證頭和 token 刷新

## 架構規則

### 前端架構
- 使用 Vue 3 + Composition API
- 狀態管理：Pinia (stores 目錄)
- UI 組件：Ant Design Vue
- 路由：Vue Router 4

### 後端架構
- Node.js + Express
- 資料庫：MySQL 8.0 (Docker)
- ORM：原生 SQL 查詢
- 認證：JWT Token

## 代碼規範

### 命名規則
- 函數命名：使用 `handleXXX` 格式（如 `handleClick`）
- 組件命名：PascalCase
- 文件命名：kebab-case 或 camelCase

### 錯誤處理
- 後端：使用統一的錯誤處理中間件
- 前端：使用 try-catch 和 message 提示
- 調試：優先使用 console.error 而不是 logger

## 部署規則

### Docker 服務
- MySQL：使用 Docker 容器
- 應用：可選擇 Docker 部署或直接部署
- 環境變數：通過 .env 文件管理

### 安全規則
- 密碼：使用 bcryptjs 加密
- Token：JWT 管理，支持黑名單
- API：統一認證和權限控制 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnshaw77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
