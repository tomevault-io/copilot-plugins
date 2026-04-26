## nextjs-start-template

> Next.js 16 + React 19 项目，采用领域驱动三层架构。

# 项目规范

Next.js 16 + React 19 项目，采用领域驱动三层架构。

**输出语言**: 所有回复必须使用中文

## 架构 (三层)

```
domain/              → 业务逻辑层 (框架无关，禁止 React)
src/lib/             → 基础设施层 (HTTP、工具函数)
src/components/ui/   → 基础 UI (shadcn，禁止业务逻辑)
src/components/domain/ → 业务 UI (连接 domain 与 ui)
src/app/             → 页面路由层 (仅 page/layout/route)
```

**依赖规则**: `domain/` 可以导入 `@/lib/*`，禁止导入 `@/components/*`、`@/app/*`、`@/hooks/*`、`@/store/*`

## 关键规则

- **组件定义**: 使用 `export function Name() {}` (禁止箭头函数，仅限组件导出，回调/闭包允许箭头函数)
- **类型定义**: 优先使用 `type` (非 interface)

## 数据流模式

```
组件 (useQuery) → Domain Controller (静态方法) → Service → HTTP
```

- 组件层使用 `useQuery` 管理状态，调用 `Controller.method(http, options)`
- HTTP 实例从 `@/service/index.base` 导入并注入
- Query Key 必须来源于 Domain 层常量 (`domain/{module}/const/api.ts`)

## 测试策略

- **domain 层**: 优先编写单元测试（纯逻辑）
- **UI 层**: 主要进行集成测试

## 参考文档

| 任务     | 文档                           |
| -------- | ------------------------------ |
| 架构详情 | @docs/architecture.md          |
| 目录约定 | @docs/conventions/directory.md |
| 命名规范 | @docs/conventions/naming.md    |
| 编码规范 | @docs/conventions/coding.md    |

## 路径别名

- `@/*` → `src/*`
- `@domain/*` → `domain/*`

## 技术栈

Next.js 16 (App Router) | React 19 | TypeScript 5.9 | Tailwind CSS 4 | Zustand | TanStack Query | ky | Vitest + MSW

## 命令

```bash
pnpm dev          # 开发服务器 (port 5373)
pnpm lint:fix     # ESLint 修复
pnpm test:run     # 运行测试
```

## Git 钩子

- **commit-msg**: commitlint 校验（angular preset，类型见 `commitlint.config.js`）
- **pre-commit**: lefthook 自动执行 `lint:fix` + `vitest --changed`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kkfive) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
