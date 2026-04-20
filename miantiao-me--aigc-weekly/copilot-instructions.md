## aigc-weekly

> - **语言**：所有对话、提交信息与界面交互均使用简体中文

# AGENTS.md

为 AI 代码助手提供本仓库的开发指南。

## 全局约定

- **语言**：所有对话、提交信息与界面交互均使用简体中文
- **环境**：Node.js 22 + pnpm 10；脚本假定 `pnpm` 在 PATH 中
- **工作目录**：仓库根目录；临时文件写入 `data/`、`.next/` 或 `.wrangler/`
- **忽略**：`agent/.claude` 目录中的文件

## 常用命令

```bash
# 开发
pnpm dev              # Next.js 开发服务器 (localhost:3000)
pnpm dev:worker       # Worker 开发 (localhost:2442)

# 构建与部署
pnpm build            # 生产构建
pnpm deploy           # 完整部署（数据库迁移 + 应用）
pnpm deploy:worker    # 部署 Worker

# 工具
pnpm generate:types   # 生成 Cloudflare + Payload 类型定义
pnpm lint:fix         # 代码检查并自动修复（优先使用）
pnpm payload          # Payload CLI（迁移等）
pnpm clean            # 清理构建产物

# 单文件操作
pnpm eslint path/to/file.ts           # 检查
pnpm eslint --fix path/to/file.ts     # 修复
```

## 代码风格

使用 [@antfu/eslint-config](https://github.com/antfu/eslint-config)，配置了 pre-commit hooks 自动检查。

### 导入顺序

```typescript
// 1. 类型导入（使用 import type）
import type { CollectionConfig } from 'payload'

// 2. Node.js 内置模块（带 node: 前缀）
import path from 'node:path'

// 3. 第三方库
import { getPayload } from 'payload'

// 4. 项目内部模块（使用 @/ 别名）
import { siteConfig } from '@/lib/config'
```

### 格式规范

| 规则   | 标准                                       |
| ------ | ------------------------------------------ |
| 缩进   | 2 空格                                     |
| 引号   | 单引号                                     |
| 分号   | 不使用                                     |
| 尾逗号 | 多行时使用                                 |
| 控制台 | 禁止 `console.log`，允许 `info/warn/error` |

### 命名规范

```typescript
// 组件/类型/接口：PascalCase
export function ThemeSwitcher() {}
interface WeeklyListParams {}

// 函数/变量：camelCase
const getPayloadClient = cache(async () => {})

// 常量：SCREAMING_SNAKE_CASE
const DEFAULT_PAGE_SIZE = 3

// 文件名
// - 组件：PascalCase.tsx (ThemeSwitcher.tsx)
// - 工具/配置：camelCase.ts (config.ts)
// - 集合：PascalCase.ts (Weekly.ts)
```

### TypeScript 规范

```typescript
// 优先类型推断，避免冗余标注
const page = normalizePositiveInteger(params.page, 1)

// 复杂函数建议显式返回类型
export async function getWeeklyList(): Promise<WeeklyListResult> {}

// 禁止使用
// - `as any`、`@ts-ignore`、`@ts-expect-error`
// - 空 catch 块
```

### React/Next.js 规范

```tsx
// Server Component（默认）- 直接 async
export default async function HomePage() {
  const data = await fetchData()
  return <Component data={data} />
}

// Client Component - 文件顶部标记
'use client'

// Props 解构
export default function Layout(props: { children: ReactNode }) {
  const { children } = props
  return <>{children}</>
}
```

### 错误处理

```typescript
// 早期返回
if (!slug)
  return null

// 验证函数返回 true 或错误信息
validate: value => pattern.test(value) || '格式不正确'

// 可选链安全访问
doc.tags?.map(t => t.value) ?? []
```

## 架构概览

```
aigc-weekly/
├── app/                    # Next.js App Router
│   ├── (frontend)/         # 前台页面
│   └── (payload)/          # Payload Admin
├── agent/                  # Claude Agent（运行在 Cloudflare Containers）
├── worker/                 # Cloudflare Worker
├── collections/            # Payload CMS 数据模型
├── components/             # React 组件
├── lib/                    # 工具函数与配置
├── migrations/             # 数据库迁移（自动生成，勿修改）
└── public/                 # 静态资源
```

### 关键文件

| 文件                | 说明         |
| ------------------- | ------------ |
| `payload.config.ts` | CMS 核心配置 |
| `next.config.mjs`   | Next.js 配置 |
| `eslint.config.mjs` | ESLint 配置  |

## 本地开发

```bash
pnpm install          # 安装依赖
pnpm generate:types   # 生成类型（必须）
pnpm dev              # 启动开发服务器
# 前台：http://localhost:3000
# 后台：http://localhost:3000/admin
```

## 部署

- 平台：Cloudflare (Workers + D1 + R2)
- 构建：OpenNext，输出到 `.open-next`
- 命令：`pnpm deploy` 自动执行数据库迁移并部署

## ESLint 忽略

- `migrations/**` - 自动生成的迁移文件
- `*types.ts` - 生成的类型定义
- `**/*.d.ts` - TypeScript 声明文件
- `**/importMap.js` - Payload 导入映射

---
> Source: [miantiao-me/aigc-weekly](https://github.com/miantiao-me/aigc-weekly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
