## ai-chatbox

> 1. **路由使用文件夹**: TanStack Start 子路由使用文件夹结构，如 `routes/settings/index.tsx`、`routes/settings/model-providers.tsx`，不使用 `routes/settings.index.tsx`、`routes/settings.model-providers.tsx` 这样的扁平命名。

# 项目开发规范

## 文件结构

1. **路由使用文件夹**: TanStack Start 子路由使用文件夹结构，如 `routes/settings/index.tsx`、`routes/settings/model-providers.tsx`，不使用 `routes/settings.index.tsx`、`routes/settings.model-providers.tsx` 这样的扁平命名。
2. **路由目录只放路由文件**: `routes/` 下优先只放会导出 `Route` 的路由入口；页面组件、类型和工具放到 `src/features/<feature>/`，避免被 TanStack Router 扫描成路由。
3. **组件文件按功能拆分**: 页面内的子组件放在对应功能目录下，如：
   ```
   features/
     chat/
       sidebar.tsx        # 侧边栏组件
       message-list.tsx   # 消息列表组件
       chat-composer.tsx  # 输入框组件
       settings-dialog.tsx # 设置弹窗组件
       types.ts           # 类型定义
       utils.ts           # 工具函数
   routes/
     index.tsx          # 路由入口（默认导出 Route）
   ```
4. **共享类型**: 路由内共享的类型放在 `types.ts` 中。

## 组件开发

1. **优先使用 shadcn/ui 组件**: 开发页面 UI 时，必须优先查找和使用 shadcn/ui 组件库中的现有组件。
2. **自定义样式仅作补充**: 只有当 shadcn/ui 组件无法满足功能需求时，才可以使用 Tailwind CSS 编写自定义样式。
3. **缺少组件时自动安装**: 如果需要的 shadcn/ui 组件尚未安装，使用以下命令自动添加：
   ```bash
   bun x shadcn@latest add <component-name> -c packages/ui
   ```
   shadcn 组件安装在 `packages/ui/src/components` 目录下，导入时使用：
   ```tsx
   import { ComponentName } from '@workspace/ui/components/component-name'
   ```
4. **优先使用 lucide 图标**: 本项目的图标库是 lucide-react，需要图标时优先从 lucide 选取。

## 技术栈

- 框架: React 19 + TanStack Start
- 样式: Tailwind CSS v4
- 组件库: shadcn/ui (base-nova style)
- 图标: lucide-react
- 构建: Vite + Nitro (Cloudflare Workers)
- 数据库: Drizzle ORM + Cloudflare D1

---
> Source: [langliu/ai-chatbox](https://github.com/langliu/ai-chatbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
