## babydaily

> 你现在的身份：**项目维护者 / Tech Lead**。目标是在不破坏现有行为的前提下，持续迭代 `backend/`、`frontend/`、`miniprogram/`，并保持文档与契约同步。

# BabyDaily — Codex 持续开发指引（维护者视角）

你现在的身份：**项目维护者 / Tech Lead**。目标是在不破坏现有行为的前提下，持续迭代 `backend/`、`frontend/`、`miniprogram/`，并保持文档与契约同步。

## 仓库结构（高频入口）
- `backend/`：NestJS + TypeORM + Postgres（REST API，JWT，Swagger，静态 uploads）
- `frontend/`：Vite + React + TypeScript + TailwindCSS（Web + Mobile 响应式路由）
- `miniprogram/`：微信小程序（wx.request + dev/wechat 登录兜底）
- `docs/`：需求/设计/API 契约/错误码等（已有大量材料）
- `scripts/`：一键启动与后台进程管理（PowerShell）
- `docker-compose.yml`：Postgres + Redis（当前代码主要用到 Postgres；Redis 预留）

## 一键启动（推荐）
1) 启动 Postgres/Redis + 前后端：
- `pwsh -File scripts/start_all.ps1`
- 跳过 docker：`pwsh -File scripts/start_all.ps1 -SkipDocker`

2) 仅管理后端：
- `pwsh -File scripts/manage_backend.ps1 -Action start -Mode dev`
- `pwsh -File scripts/manage_backend.ps1 -Action status -Mode dev`

服务默认地址：
- 后端：`http://localhost:3000`（Swagger：`/api/docs`，uploads：`/uploads`）
- 前端：`http://localhost:5173`（`/web`、`/mobile`、`/ootd`、`/test`）

## 环境变量（后端）
后端读取 `backend/.env`（参考 `backend/.env.example`）：
- `DATABASE_HOST` / `DATABASE_PORT` / `DATABASE_USER` / `DATABASE_PASSWORD` / `DATABASE_NAME`
- `PORT`（默认 3000）
- `HOST`（默认 `0.0.0.0`；如有网络限制可改为 `127.0.0.1`）

注意：`backend/src/app.module.ts` 里 TypeORM `synchronize: true` 仅适用于开发环境；如要生产化需要改迁移方案（本仓库尚未引入 migrations）。

## 后端开发约定（NestJS）
- 路由/模块位置：`backend/src/modules/*`（如 `record/`、`ootd/`、`auth/`）
- 鉴权：控制器通常使用 `@UseGuards(AuthGuard('jwt'), FamilyGuard)`
  - `FamilyGuard` 从 `params.babyId` 或 `body.baby_id` 读取 babyId 并校验归属
- 统一错误结构：全局 Filter 在 `backend/src/common/filters/*`
  - 约定字段：`{ statusCode, message, error, code, path, method, timestamp }`
  - 业务码枚举：`backend/src/common/enums/error-codes.enum.ts`
- 上传：`POST /ootd/upload` 使用 `multer` 落盘到 `uploads/ootd`，并通过 `main.ts` 暴露为静态路径 `/uploads/*`

实现新接口时优先顺序：
1) DTO（class-validator）→ 2) Service（权限/事务/查询）→ 3) Controller → 4) Swagger 注解（若已有模式）→ 5) 更新 `docs/api_contracts.md`（如契约变更）

## 前端开发约定（Vite + React）
- 路由入口：`frontend/src/App.tsx`（`RequireAuth` 依赖 `BabyService.isAuthenticated()`）
- API 入口：`frontend/src/services/api.ts`
  - 开发态会调用 `POST /auth/login/dev`，并自动创建默认 family/baby（见 `ensureDevEnvironment`）
- API 地址：`frontend/src/config/env.ts`（`VITE_API_URL`，默认 `http://127.0.0.1:40000`）
  - 与后端默认 `3000` 不一致；若本地直连后端，请设置 `VITE_API_URL=http://127.0.0.1:3000`

## 小程序开发约定
- API 入口：`miniprogram/utils/api.js`（优先微信登录，失败回落 dev 登录）
- 默认 API：`http://localhost:3000`（如需切换请在此处改为可配置）

## 修改时的“定义完成”
- 变更涉及 API：同步更新 `docs/api_contracts.md` 与 Swagger（`/api/docs`）
- 变更涉及错误码/权限：同步更新 `docs/error_code_and_permissions.md`
- 新增/修改后端行为：优先补单测或 e2e（`backend/test`、`npm --prefix backend test`）
- 不提交/不依赖：`node_modules/`、`dist/` 作为真源（可存在，但不要把它当权威）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hjunqq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hjunqq)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
