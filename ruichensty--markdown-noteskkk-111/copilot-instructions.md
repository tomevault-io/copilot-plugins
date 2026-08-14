## markdown-noteskkk-111

> Markdown Notes — 基于 React 19 + TypeScript + IndexedDB 的 Markdown 笔记应用。

# AGENTS.md

## 项目概述

Markdown Notes — 基于 React 19 + TypeScript + IndexedDB 的 Markdown 笔记应用。

## 常用命令

- `pnpm dev` — 启动开发服务器 (port 3000)
- `pnpm build` — 生产构建
- `pnpm type-check` — TypeScript 类型检查
- `pnpm lint` — ESLint 检查
- `pnpm lint:fix` — ESLint 自动修复
- `pnpm format` — Prettier 格式化

## 架构

- `src/components/` — UI 组件
- `src/hooks/` — 自定义 Hooks
- `src/utils/` — 工具函数（IndexedDB 操作、导出）
- `src/context/` — React Context（Theme、Toast）
- `src/types/` — TypeScript 类型定义
- 存储层：IndexedDB（4 个 store: notes, folders, settings, files）

## 代码规范

- 组件使用函数式组件 + memo 优化
- 路径别名：`@components`, `@hooks`, `@types`, `@utils`, `@context`
- CSS：Tailwind CSS + 自定义 CSS 变量（HSL 色彩系统）
- 不添加注释除非明确要求
- 运行 type-check 和 lint 通过后再提交

---
> Source: [ruichensty/markdown-notesKKK_111](https://github.com/ruichensty/markdown-notesKKK_111) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
