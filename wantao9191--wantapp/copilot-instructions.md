## project-structure

> 项目结构和架构指南


# 项目结构指南

这是一个基于 Next.js 15 + TypeScript + Ant Design 的全栈管理系统项目。

## 核心架构

- **前端框架**: Next.js 15 with App Router
- **UI 库**: Ant Design 5.x + UnoCSS
- **数据库**: PostgreSQL + Drizzle ORM
- **认证**: JWT + NextAuth
- **测试**: Vitest + Testing Library

## 目录结构

- `src/app/` - Next.js App Router 页面和 API 路由
- `src/components/` - 可复用组件
- `src/lib/` - 工具函数和配置
- `src/hooks/` - 自定义 React Hooks
- `src/types/` - TypeScript 类型定义
- `src/db/` - 数据库相关代码
- `tests/` - 测试文件
- `scripts/` - 构建和数据库脚本

## 路径别名

使用 `@/` 作为 `src/` 的别名：
- `@/components/*` → `./src/components/*`
- `@/lib/*` → `./src/lib/*`
- `@/hooks/*` → `./src/hooks/*`
- `@/types/*` → `./src/types/*`

## 关键文件

- [package.json](mdc:package.json) - 项目依赖和脚本
- [next.config.js](mdc:next.config.js) - Next.js 配置
- [tsconfig.json](mdc:tsconfig.json) - TypeScript 配置
- [src/middleware.ts](mdc:src/middleware.ts) - 认证中间件
- [src/app/layout.tsx](mdc:src/app/layout.tsx) - 根布局组件

---
> Source: [wantao9191/wantapp](https://github.com/wantao9191/wantapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
