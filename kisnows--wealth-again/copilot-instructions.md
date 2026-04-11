## wealth-again

> 本文件为 AI 助手提供上下文，以便更好地理解 "Wealth Again" 项目。它概述了项目的目标、架构、技术和开发规范。

# GEMINI.md - AI 助手上下文文件

本文件为 AI 助手提供上下文，以便更好地理解 "Wealth Again" 项目。它概述了项目的目标、架构、技术和开发规范。

## 1. 项目概览

**Wealth Again** 是一个全面的个人财务管理平台，旨在提供资产、负债、收入和税务的一体化视图。

- **核心目标**: 作为一个多币种的个人财务中心，提供详细的追踪、分析和预测功能。
- **主要功能**:
    - 支持多币种账户管理和实时货币换算。
    - 详细的收入和税务计算，支持多种收入类型（工资、奖金、股权）和特定地区规则（例如中国的社保和税法）。
    - 包含历史估值的资产和负债追踪。
    - 用于概览净资产、收入和支出的财务仪表盘。
    - 用于管理特定城市税务和社保配置的规则系统。
    - 为管理员提供用户管理和模拟登录以进行支持的能力。

### 架构与技术

- **框架**: 基于 **Next.js 15** (App Router) 构建的全栈应用。
- **语言**: **TypeScript**。
- **数据库**: 使用 **Prisma** 作为 ORM。开发环境使用 **SQLite**，生产环境计划使用 PostgreSQL。详细的 schema 定义于 `prisma/schema.prisma`。
- **前端**:
    - **UI**: React, **Tailwind CSS**, 和 **shadcn/ui**。
    - **数据请求**: **SWR** 是所有客户端数据请求的标准，API hooks 定义在 `src/lib/api/` 中。
    - **状态管理**: **Zustand** 用于共享的全局客户端状态 (`src/lib/state/`)。
- **后端**:
    - **API**: 在 `src/app/api/` 中使用 Next.js Route Handlers 实现 RESTful API。
    - **服务**: 业务逻辑被封装在位于 `src/server/services/` 的服务层中。
    - **认证**: **better-auth** 处理凭证登录、会话与 Cookie。
- **测试**: **Vitest** 是用于单元和集成测试的框架。测试文件位于 `src/tests/`。
- **工具**:
    - **包管理器**: `pnpm`。
    - **代码检查与格式化**: `Biome`。

## 2. 构建与运行

以下命令定义在 `package.json` 中，是开发任务的主要入口。

### 初始化设置

1.  **安装依赖**:
    ```bash
    pnpm install
    ```
2.  **设置数据库**:
    创建本地 SQLite 数据库并应用迁移。
    ```bash
    pnpm prisma migrate dev
    ```
3.  **填充种子数据**:
    为开发环境填充初始数据。
    ```bash
    # 种子数据命令定义在 package.json 的 "prisma" 部分
    # 通常通过 prisma CLI 运行，但也可以直接运行。
    node prisma/seed.js
    ```

### 常用命令

- **运行开发服务器**:
  在开发模式下启动 Next.js 应用（使用 Turbopack），监听 `http://localhost:4000`。
  ```bash
  pnpm dev
  ```

- **运行测试**:
  使用 Vitest 执行测试套件。
  ```bash
  pnpm test
  ```

- **代码检查与格式化**:
  使用 Biome 检查代码质量和格式问题。
  ```bash
  # 检查问题
  pnpm lint

  # 自动修复问题
  pnpm format
  ```

- **构建生产版本**:
  创建应用的生产就绪版本。
  ```bash
  pnpm build
  ```

## 3. 开发规范

本项目遵循一系列严格的规范，主要记录在 `doc/` 目录中。

- **单一事实来源 (Single Source of Truth)**: `doc/` 目录是所有产品 (`prd.md`)、技术 (`tech.md`) 和前端 (`frontend-spec.md`) 规范的最终来源。任何重要的功能变更都必须在此处反映。
- **样式**: 只能使用 **Tailwind CSS** 和 **shadcn/ui** 进行样式设计，不应引入其他样式方案。
- **目录结构**:
    - `src/app/**`: Next.js 路由和页面布局。
    - `src/components/ui/**`: 未经修改的 `shadcn/ui` 组件。
    - `src/components/modules/**`: 可复用的、特定领域的业务组件 (例如 `AccountTable`)。
    - `src/lib/api/**`: 客户端数据请求钩子 (使用 SWR)。
    - `src/lib/domain/**`: 纯粹的、客户端的业务逻辑和工具函数。
    - `src/server/services/**`: 后端业务逻辑 (例如 `income.ts`, `ledger.ts`)。
- **状态管理**: 对跨组件的少量客户端状态使用 **Zustand**。服务端状态应通过 **SWR** 管理。
- **提交与合并请求 (Commits & PRs)**: (TODO: 如果有，可从 git 历史或贡献指南中推断)。

## 4. 交互规则

- 请始终使用中文回答所有问题。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kisnows)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kisnows)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
