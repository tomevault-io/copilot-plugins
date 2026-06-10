## interview-vue-fastapi

> 这是一个 30 分钟 AI 实战面试用的全栈 Starter。

# 项目指令

这是一个 30 分钟 AI 实战面试用的全栈 Starter。

## 技术栈

- 前端：Vue 3 + Vite + TypeScript + Vue Router + Pinia + Axios + Element Plus
- 后端：FastAPI + SQLModel + MySQL + Redis
- 部署：Docker Compose 可选

## 优先级

1. 先保证 PRD 的核心业务流程能跑通
2. API 契约清楚
3. 前后端真实联通
4. 构建和测试通过
5. 文档同步更新
6. UI 简洁可用，不追求像素级还原

## 面试流程

拿到 PRD 和 HTML 原型后：

1. 先阅读 PRD 和原型，不要直接写代码。
2. 填写 `docs/PRD_INTAKE.md`，提取 MVP 范围、主资源、字段、页面、接口。
3. 更新 `docs/API_DESIGN.md`，先定 API 契约。
4. 把现有 `Item` CRUD 骨架改成题目的主业务资源。
5. 后端改 model/schema/service/router/test。
6. 前端改 types/services/form/list/filter，优先使用 Element Plus 组件。
7. 跑后端编译、后端测试、前端 build、健康检查。
8. 更新 `docs/DELIVERY_NOTES.md` 和 `docs/TEST_CHECKLIST.md`。

HTML 原型只表达交互意图，不要求像素级还原。

## 常用命令

前端：

- `cd frontend && npm install`
- `cd frontend && npm run dev -- --host 127.0.0.1 --port 5174`
- `cd frontend && npm run build`

后端：

- `cd backend && uv sync`
- `cd backend && uv run uvicorn app.main:app --reload --port 8000`
- `cd backend && uv run python -m compileall app`
- `cd backend && uv run pytest`

Docker：

- `docker compose up --build`

## 当前本地服务

- 后端：`http://127.0.0.1:8000`
- 前端：`http://127.0.0.1:5174`
- MySQL：`127.0.0.1:3306`，数据库 `interview_vue_fastapi`
- Redis：`127.0.0.1:6379`

## 编码规则

- Vue 组件里不要直接写 fetch/axios，请走 `frontend/src/services`。
- UI 控件优先使用 Element Plus 和 `@element-plus/icons-vue`。
- 共享类型放在 `frontend/src/types`。
- 路由放在 `frontend/src/router`。
- 全局状态放在 `frontend/src/stores`。
- 后端路由统一放在 `/api` 下。
- API 变化必须同步更新 `docs/API_DESIGN.md`。
- 不要为了小题临时引入重依赖。

---
> Source: [zgm2003/interview-vue-fastapi](https://github.com/zgm2003/interview-vue-fastapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
