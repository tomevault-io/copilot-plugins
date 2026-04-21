## quiz-monorepo

> 本文档为 AI 助手提供项目核心开发规范。

# AI 开发指南

本文档为 AI 助手提供项目核心开发规范。

## 项目概述

Quiz Monorepo - 开发者技术问答应用，前后端分离架构：

- **前端** (`apps/quiz-app`): Vue 3 + Vite + TypeScript
- **管理后台** (`apps/quiz-admin`): Vue 3 + Element Plus + UnoCSS
- **后端** (`apps/quiz-backend`): NestJS + Prisma 7 + MySQL
- **UI 库** (`packages/ui`): 共享组件 + Storybook

| 层级     | 技术                                        |
| -------- | ------------------------------------------- |
| 前端     | Vue 3 Composition API + Vite + Pinia        |
| 管理后台 | Vue 3 + Element Plus + UnoCSS + Pinia       |
| 后端     | NestJS + Prisma 7 + MySQL (MariaDB adapter) |
| 样式     | SCSS + UnoCSS (Tailwind 4)                  |
| 测试     | Vitest/Jest (单元) + Cypress (E2E)          |
| 包管理   | pnpm workspace + Turborepo                  |

## 核心规范

### 代码风格

- **注释**：所有代码必须添加**中文注释**（函数用 JSDoc，复杂逻辑加行内说明）
- **TypeScript**：完善类型声明，避免 `any`（必要时需注释说明）
- **Vue 组件**：使用 `<script setup lang="ts">`，Props/Emits 必须声明类型
- **命名**：组件 PascalCase，函数 camelCase，常量 UPPER_SNAKE_CASE，CSS BEM 或 kebab-case

### 样式规范

偏好 SCSS + UnoCSS `@apply` 指令，BEM 命名：

```scss
.button {
  @apply flex items-center gap-2 px-4 py-2 rounded-lg;
  &--disabled {
    @apply cursor-not-allowed opacity-50;
  }
}
```

布局要点：全屏高度用 flex 链式传递（`min-h-screen flex flex-col` → `flex-1`）；深浅模式色阶区分（浅色 300 级，深色 500 级）。

## 常用命令

```bash
# 开发
pnpm dev              # 启动全部 (前端 10000, 后端 10020, UI 10030)
pnpm dev:frontend     # 仅前端
pnpm dev:backend      # 仅后端
pnpm -C apps/quiz-admin dev  # 管理后台 (10050)

# 代码质量检查
pnpm run check        # 快速检查：lint + type-check + test:unit (~5s)
pnpm run check:e2e    # 完整检查：包含 E2E 测试 (~5min)
pnpm lint:fix         # 自动修复代码格式问题

# 数据库管理（后端目录下）
pnpm -C apps/quiz-backend run migrate:deploy:dev   # 应用迁移到开发库
pnpm -C apps/quiz-backend run migrate:status       # 查看所有环境迁移状态
pnpm -C apps/quiz-backend run db:studio            # 数据库可视化
pnpm -C apps/quiz-backend run db:seed:dev          # 插入开发数据
pnpm -C apps/quiz-backend run db:reset:test        # 重置测试数据

# 测试
pnpm test:unit              # 单元测试 (~885 tests, ~10s)
pnpm test:unit:coverage     # 单元测试 + 覆盖率报告
pnpm test                   # 完整测试（包括 E2E，~5 分钟）
```

更多脚本说明见 [apps/quiz-backend/scripts/README.md](apps/quiz-backend/scripts/README.md)

## Prisma 7 特殊说明

- **配置文件**：`apps/quiz-backend/prisma.config.ts`（不在 schema.prisma 中配置 url）
- **Placeholder URL**：`process.env.DATABASE_URL ?? "placeholder"` 确保 generate 不依赖真实数据库
- **迁移限制**：`prisma migrate dev` 需 shadow DB 权限（RDS 无），手动写 SQL + `prisma migrate deploy`
- **环境变量**：两层策略 — `.env.{env}` 提交到仓库（端口等），`.env.{env}.local` 不提交（数据库密码等）
- **新成员上手**：clone → 复制 `.env.{env}.example` 到 `.env.{env}.local` → 填密码 → `pnpm dev`

## Git 规范

- **Commit**：Conventional Commits（feat / fix / docs / refactor / test / chore）
- **Hooks**：pre-commit lint-staged；pre-push type-check + test:unit
- **分支**：简单修改直接 main，功能/Bug 创建 feature 分支 → PR

## 测试策略

| 包           | 单元测试                         | E2E 测试                                |
| ------------ | -------------------------------- | --------------------------------------- |
| quiz-app     | Vitest (~122 tests)              | Cypress (6 spec, Mock API, ~33 tests)   |
| quiz-admin   | Vitest (~247 tests)              | Cypress (11 spec, 真实后端, ~129 tests) |
| quiz-backend | Jest (~370 tests, 86~95% 覆盖率) | -                                       |
| ui           | Vitest (~224 tests)              | Playwright (Storybook, 10 story)        |

日常：`pnpm test:unit`（~5 秒）| PR 前：`pnpm test`（~5 分钟，含 E2E）

## 常见问题

```bash
pnpm clean:ports                              # 端口占用
pnpm -C apps/quiz-backend run prisma:generate # Prisma Client 缺失
pnpm install && pnpm -C apps/quiz-backend run build  # 依赖问题
```

### UnoCSS 图标生产构建不加载

**现象**：开发环境图标正常，生产构建后所有 `i-carbon-*` 图标不显示，构建日志出现 `[unocss] failed to load icon "carbon-*"`。

**原因**：pnpm 严格依赖隔离下，UnoCSS `presetIcons` 的自动发现机制无法找到 `@iconify-json/carbon` 包（即使已安装）。这是 pnpm monorepo 的已知问题（[unocss#2905](https://github.com/unocss/unocss/issues/2905)）。

**解决方案**：在 `uno.config.ts` 中显式导入并传入图标集合，不依赖自动发现：

```ts
import { icons as carbonIcons } from "@iconify-json/carbon";

presetIcons({
  scale: 1.2,
  warn: true,
  collections: {
    carbon: () => carbonIcons,
  },
}),
```

> **注意**：`.npmrc` 中的 `public-hoist-pattern[]=@iconify-json/*` 不足以解决此问题，必须显式导入。

## 内容审查规范

本项目涉及跨仓库内容生产（VitePress 笔记、Slidev 幻灯片、Quiz 题目），审查/对比第三方库官方文档时必须遵循以下流程：

### 审查流程

1. **WebFetch 首页** → 获取官方文档的完整站点导航（所有页面链接）
2. **逐页 WebFetch** → 全部页面过一遍（通常 3-8 页），获取完整的一手信息
3. **context7 补充** → 适合快速查核心用法，但边缘页面（troubleshoot、migration）可能缺失，不能替代逐页浏览
4. **本地验证** → 在项目中实际检查（`ls`、`cat` 关键文件），确认当前版本的真实行为
5. **交叉比对后再动手** → 只有一手文档 + 本地验证都支持的情况下，才能判定现有内容有误

### 禁止事项

- **禁止把"文档摘要未提及"等同于"该特性已废弃"**
- **禁止在未经本地验证的情况下修改用户已有的正确内容**
- **禁止仅凭 context7 或 AI 总结就下"过时"结论**

### 跨仓库工作目录

统一在本仓库（quiz-monorepo）对话，跨目录操作其他项目：

| 项目           | 路径                                            |
| -------------- | ----------------------------------------------- |
| VitePress 笔记 | `/Users/zhangxu/workspace/IllegalCreedWebsite/` |
| Slidev 幻灯片  | `/Users/zhangxu/workspace/SlideStack/`          |
| Quiz 题目      | 本仓库 `apps/quiz-backend/prisma/content/`      |

## 相关文档

- [docs/product.md](./docs/product.md) - 产品需求 + 路线图
- [docs/dev.md](./docs/dev.md) - 技术架构 + 待实现功能详细计划
- [docs/plans/](./docs/plans/) - 方案设计文档（实施前先写方案，规则详见 dev.md "计划文档" 章节）；**文件名必须以 `YYYYMMDD-` 开头**（如 `20260331-xxx.md`），便于按时间排序

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IllegalCreed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
