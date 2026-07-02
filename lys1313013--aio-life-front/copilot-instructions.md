## aio-life-front

> AIO Life 是一个一站式生活管理系统，用于记录、追踪和分析个人生活数据。基于 **Vue Vben Admin v5.5.9** 的 **Ant Design Vue** 变体构建，采用 monorepo 架构。


## 项目概述

AIO Life 是一个一站式生活管理系统，用于记录、追踪和分析个人生活数据。基于 **Vue Vben Admin v5.5.9** 的 **Ant Design Vue** 变体构建，采用 monorepo 架构。

## 技术栈

- **包管理器**: pnpm 10.22.0（monorepo + workspaces）
- **构建编排**: Turborepo
- **框架**: Vue 3 + TypeScript + Vite 7
- **UI 组件库**: Ant Design Vue 4.x
- **样式**: Tailwind CSS
- **图表**: ECharts
- **状态管理**: Pinia
- **测试**: Vitest（单元测试）、Playwright（端到端测试）
- **代码检查**: ESLint、Stylelint、Prettier、Commitlint、cspell
- **Git Hooks**: Lefthook

## 常用命令

```bash
pnpm run dev            # 启动开发服务器
pnpm run build          # 生产环境构建
pnpm run lint           # 代码检查
pnpm run format         # 代码格式化
pnpm run typecheck      # 类型检查
pnpm run check          # 全量检查（循环依赖、依赖、类型、拼写）
pnpm run test:unit      # 运行单元测试
pnpm run test:e2e       # 运行端到端测试
```

## Monorepo 目录结构

- `apps/web-antd/` — 主应用（Ant Design Vue 版）
- `packages/` — 共享包（stores、utils、locales、styles、types、icons）
- `packages/@core/` — 核心框架包（基础组件、UI kits、composables、preferences）
- `packages/effects/` — 效果包（layouts、plugins、hooks、request、access）
- `internal/` — 内部工具（vite-config、tailwind-config、tsconfig、lint-configs）
- `scripts/` — 构建/开发脚本

## 应用架构（`apps/web-antd/src/`）

### API 层（`api/core/`）

- 使用 `#/api/request` 中的 `requestClient`（基于 `@vben/request` 配置的请求客户端）
- API 函数均为具名 async 导出，CRUD 操作使用 `requestClient.post()`，读取使用 `requestClient.get()`
- 响应格式：`{ rscode: '0', data: ... }`，成功码为 `'0'`
- 注意：后端返回的 ID 为 **string** 类型

### 路由（`router/routes/modules/`）

- 每个模块有独立的路由文件（如 `dashboard.ts`、`my-hub.ts`、`time-management.ts`）
- 路由使用懒加载：`component: () => import('#/views/...')`
- `#` 别名指向 `apps/web-antd/src/`

### 状态管理（`store/`）

- 使用 Pinia store 管理认证及业务状态（如 `password-vault.ts`）

## 编码规范

1. **适配暗色模式** — 避免将背景色和文字颜色硬编码为纯黑或纯白
2. **适配移动端** — 所有页面需支持移动端展示
3. **遵守 ESLint 规则** — 遵循现有 lint 配置
4. **接口 loading 效果** — 调用后端接口时必须有 loading 效果
5. **后端 ID 为 string** — 后端返回的 ID 类型为 string，不要当作 number 使用
6. 文字和样式要尽可能的简洁
7. 确认弹窗尽可能在按钮旁边弹出
8. 编辑弹窗要上下居中（出一些场景要跟手外），编辑弹窗可以没有 title

---
> Source: [lys1313013/aio-life-front](https://github.com/lys1313013/aio-life-front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
