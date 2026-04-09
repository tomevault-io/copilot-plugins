## openspec2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

出缺勤管理系統（Attendance Management System） — 全棧 Web 應用，符合台灣勞基法規範。

**技術棧**: Vue 3 + TypeScript (前端) / Express.js + TypeScript (後端) / MongoDB / Docker Compose

## 開發環境啟動

```bash
# 首次啟動
cp .env.example .env
docker compose up -d
docker compose exec backend npm run seed  # 建立預設 admin 帳號

# 日常開發
docker compose up -d
docker compose logs -f backend   # 查看後端日誌
docker compose logs -f frontend  # 查看前端日誌
docker compose down              # 停止所有服務
```

服務地址：
- 前端: http://localhost:5173
- 後端 API: http://localhost:3000/api
- Swagger UI: http://localhost:3000/api-docs（需 `.env` 中設定 `ENABLE_SWAGGER=true`）
- Mongo Express: http://localhost:8081

預設帳號: `admin@company.com` / `Admin@1234`

## 後端命令（`cd backend`）

```bash
npm run dev    # 開發模式（tsx watch）
npm run build  # TypeScript 編譯至 dist/
npm run start  # 執行編譯版本
npm run seed   # 初始化資料
```

## 前端命令（`cd frontend`）

```bash
npm run dev     # Vite 開發伺服器
npm run build   # vue-tsc + vite build
npm run preview # 預覽生產構建
```

## 架構說明

### 後端結構（`backend/src/`）

```
config/database.ts          # MongoDB Mongoose 連接
middleware/auth.ts           # JWT 驗證 + RBAC 角色中間件
models/                      # Mongoose Schema（User, Attendance, LeaveRequest, LeaveBalance, OvertimeRequest, WorkSchedule, RefreshToken）
routes/                      # API 路由控制器（一個文件對應一個資源）
services/
  leaveService.ts            # 假別計算（台灣勞基法年資對應天數）
  emailService.ts            # Email 通知（未設定 SMTP 時降級為 console.log）
  workScheduleService.ts
scripts/seed.ts              # 初始資料腳本
index.ts                     # 應用入口，路由掛載
```

### 前端結構（`frontend/src/`）

```
utils/api.ts        # Axios 實例 + 401 自動 Token 刷新攔截器
stores/             # Pinia 狀態管理（auth.ts, user.ts）
router/index.ts     # Vue Router + 路由守衛（認證、角色、強制改密碼）
types/index.ts      # 所有 TypeScript 類型和常量（標籤對應表）
pages/              # 頁面組件（admin/, attendance/, leave/, overtime/）
components/         # 共用組件（AppLayout, AppHeader, AppSidebar, ConfirmModal, StatusBadge）
```

### 認證流程

- **JWT 雙層**：Access Token（1 小時）+ Refresh Token（7 天），均存 httpOnly Cookie
- **Token 輪轉**：刷新時舊 Token 記錄於 RefreshToken 集合（黑名單機制）
- **RBAC 角色**：`admin` / `hr` / `employee`；主管透過 User.supervisorId 標記
- **強制改密碼**：新帳號 `mustChangePassword: true`，路由守衛強制跳轉 `/change-password`

### 審核流程

請假和加班均為兩層審核：

1. **員工提交** → pending
2. **主管核准** → supervisor_approved（或 rejected）
3. **HR/Admin 核准** → approved（或 rejected）

### 假別計算（台灣勞基法）

`backend/src/services/leaveService.ts` 依年資自動計算年假天數，工作日計算參考 `WorkSchedule.workDays`。

### Vite 開發代理

前端 `/api/*` 請求代理至 `http://backend:3000`（Docker 網路名稱）。

## OpenSpec 規格文件

`openspec/` 目錄存放系統設計文件：
- `changes/attendance-management-system/` — 當前變更的設計、規格、任務
- `openspec/config.yaml` — OpenSpec 工作流配置

實作任務時參考 `openspec/changes/attendance-management-system/tasks.md`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/q1113111)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/q1113111)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
