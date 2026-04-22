## bm-md

> 本文件为 AI 编程助手（Claude Code、Cursor、Copilot 等）提供项目开发指南。

# AGENTS.md

本文件为 AI 编程助手（Claude Code、Cursor、Copilot 等）提供项目开发指南。

## 语言

始终使用 **简体中文** 进行交流和编写代码注释。使用 `context7` 查询最新 API/文档。

## 项目概述

bm.md 是一个基于 TanStack Start 和 React 的 Markdown 排版工具，支持为微信公众号、知乎专栏、掘金、网页及图片生成个性化排版。

**技术栈**: TanStack Start (React 19) | Vite 7 | Tailwind CSS 4 + shadcn/ui | TypeScript | pnpm | Vitest

## 常用命令

```bash
pnpm dev                    # 启动开发服务器 (端口 2663)
pnpm build                  # 生成 OpenAPI + 构建生产版本
pnpm lint:fix               # 自动修复 ESLint 问题

# 测试 (Vitest)
pnpm test                                              # 运行所有测试
pnpm test src/lib/markdown/extract/text.test.ts       # 运行单个测试文件
pnpm test --grep "keeps paragraph"                    # 按测试名称过滤
pnpm test --watch                                     # 监听模式

# 安装依赖 (所有依赖添加为 devDependencies)
pnpm add -D <package>

# shadcn/ui 组件
pnpm shadcn add <component>                           # 默认 registry
pnpm shadcn add shimmer-button --registry @magicui   # Magic UI 组件
```

## 项目结构

```
src/
├── components/ui/   # shadcn/ui 组件 (CLI 管理，勿手动编辑)
├── env/             # 环境变量管理 (统一入口)
├── lib/             # 核心库 (Markdown 处理、工具函数)
├── routes/          # TanStack Router 路由 (文件路由)
├── stores/          # Zustand 状态管理
├── themes/          # 主题配置 (代码高亮、Markdown 样式)
└── utils/           # 工具函数
```

## 代码风格

### 导入顺序 (自动排序)

```typescript
import { createRequire } from 'node:module' // 1. Node.js 内置
import { create } from 'zustand' // 2. 外部依赖
import { cn } from '@/lib/utils' // 3. 内部模块 (@/)
import { Fallback } from './fallback' // 4. 相对路径
```

### TypeScript

- 使用 `import type` 导入纯类型
- 接口命名 PascalCase，**禁用 `any`**
- 目标 ES2022，启用 `strictNullChecks`, `noUnusedLocals`, `noUnusedParameters`

### 格式化

| 规则       | 设置                                 |
| ---------- | ------------------------------------ |
| 缩进       | 2 空格                               |
| 引号       | 单引号                               |
| 分号       | 不使用                               |
| 尾随逗号   | 多行添加                             |
| 大括号风格 | Stroustrup (`else` / `catch` 独立行) |

### React 组件

- 组件命名 PascalCase，文件命名 kebab-case
- hooks 使用 `use` 前缀
- 已启用 `babel-plugin-react-compiler` (自动 memo 优化)

### 错误处理

- try-catch 包裹异步操作，Zod 验证外部输入
- 允许 `console.info`, `console.warn`, `console.error`

## UI 开发规范

### 核心约束

- **必须** 优先使用 Tailwind CSS / shadcn-ui 默认值 (间距、圆角、阴影)
- **必须** 使用 `cn` 工具 (`clsx` + `tailwind-merge`) 处理类名
- **必须** 使用 `Base UI` 作为可访问组件原语
- **必须** 为仅图标按钮添加 `aria-label`

### 动画

- **切勿** 在无明确要求时添加动画
- **必须** 仅对合成器属性 (`transform`, `opacity`) 进行动画
- **切勿** 对布局属性 (`width`, `height`, `margin`, `padding`) 进行动画
- 使用 `motion/react` 处理 JavaScript 动画，`tw-animate-css` 处理入场动画

### 设计

- **切勿** 使用渐变，除非明确要求
- **切勿** 使用 `h-screen`，改用 `h-dvh`
- **必须** 使用固定的 `z-index` 刻度 (不得使用任意 `z-x`)

## 路由

TanStack Router 文件路由，`src/routeTree.gen.ts` 自动生成，**勿手动编辑**。

```typescript
// API 路由
export const Route = createFileRoute('/api/upload/image')({
  server: { handlers: { POST: async ({ request }) => Response.json({ url }) } },
})

// 页面路由
export const Route = createFileRoute('/_layout/')({
  component: HomePage,
})
```

## 测试

测试文件与源文件同目录，命名 `*.test.ts`。

```typescript
import { describe, expect, it } from 'vitest'
import extract from './text'

describe('markdown -> text extract', () => {
  it('keeps paragraph separation', async () => {
    const text = await extract('First.\n\nSecond.')
    expect(text).toContain('First.')
  })
})
```

## 状态管理

Zustand + 持久化。存储名称统一使用 `bm.md.` 前缀。

```typescript
export const useEditorStore = create<EditorState>()(
  persist(set => ({ }), { name: 'bm.md.editor' }),
)
```

## 环境变量

统一通过 `src/env/index.ts` 管理：

| 前缀    | 可用范围          | 示例               |
| ------- | ----------------- | ------------------ |
| `VITE_` | 客户端 + 服务端   | `VITE_APP_URL`     |
| 无前缀  | 仅服务端 (getter) | `S3_ACCESS_KEY_ID` |

```typescript
import { env } from '@/env'

const appUrl = env.VITE_APP_URL // 客户端可用
const s3Key = env.S3_ACCESS_KEY_ID // 仅服务端
```

## 注意事项

| 禁止                           | 必须                                 |
| ------------------------------ | ------------------------------------ |
| 使用 Next.js 模式              | 提交前运行 `pnpm lint:fix`           |
| 手动修改 `src/components/ui/`  | 环境变量通过 `src/env/index.ts` 管理 |
| 引入 Lucide React 以外的图标库 | 路径别名使用 `@/` 前缀               |
| 使用 `any` 类型                | Zod 验证所有外部输入                 |
| 对布局属性进行动画             | 测试文件与源文件同目录               |

---
> Source: [miantiao-me/bm.md](https://github.com/miantiao-me/bm.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
