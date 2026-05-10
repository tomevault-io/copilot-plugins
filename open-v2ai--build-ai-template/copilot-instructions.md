## project-structure

> Build AI Template 项目结构和技术栈指南


# Build AI Template - 项目结构指南

## 项目概述

这是一个用于快速构建 AI 应用的开源模板项目，核心功能是与智能代理（Agent）进行对话。

## 技术栈

- **前端**: Next.js 15.3 + TypeScript + Tailwind CSS + Shadcn UI + React 19
- **后端**: FastAPI + Python 3.12 + SQLModel + PostgreSQL + Redis
- **AI**: OpenAI API + 自定义 Agent 框架
- **部署**: Docker + Docker Compose
- **包管理**: 前端 `pnpm`，后端 `uv`

## 核心目录结构

### 后端 (`api/`)

- **`api/app/main.py`**: FastAPI 应用入口
- **`api/app/core/`**: 核心配置 (安全, i18n)
- **`api/app/models/`**: SQLModel 数据模型
- **`api/app/schemas/`**: Pydantic 验证模式
- **`api/app/routers/`**: API 路由定义
- **`api/app/crud/`**: 数据库 CRUD 操作
- **`api/app/services/`**: 业务逻辑服务
- **`api/pyproject.toml`**: Python 依赖配置

### 前端 (`web/`)

- **`web/app/`**: Next.js App Router 页面
- **`web/components/`**: React 组件 (包括 `ui/` 和 `admin/`)
- **`web/i18n/`**: 国际化配置
- **`web/util/`**: 工具函数
- **`web/package.json`**: 前端依赖配置

## 关键配置文件

- **`deploy/docker-compose.yaml`**: 生产环境部署
- **`deploy-test/docker-compose.yaml`**: 测试环境部署

## 开发流程

1. **后端优先**: 在 `api/` 中实现模型、路由和业务逻辑。
2. **前端实现**: 在 `web/` 中创建对应的组件和页面。
3. **样式开发**: 使用 Tailwind CSS 和 Shadcn UI。
4. **国际化**: 添加翻译到 `web/app/messages/`。
5. **类型定义**: 确保前后端类型一致。

## 开发规范

- **代码注释**: 必须为所有代码添加详细的中文注释。
- **代码优先**: 专注于功能实现，让代码本身成为最好的文档。
- **注释维护**: 代码修改时必须同步更新相关注释。

## 核心功能模块

- **用户管理**: 邮箱验证码登录、权限管理
- **对话系统**: AI Agent 对话、流式响应
- **管理后台**: 用户、对话、Agent 管理
- **任务系统**: Agent 任务识别与执行

---
> Source: [open-v2ai/build-ai-template](https://github.com/open-v2ai/build-ai-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
