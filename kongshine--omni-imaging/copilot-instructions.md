## omni-imaging

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 Next.js 15 的现代 Web 应用项目，使用 TypeScript 和 Tailwind CSS 构建。项目采用了 Next.js App Router 架构模式。

## 开发命令

### 核心开发命令
- `npm run dev` - 启动开发服务器（使用 Turbopack）
- `npm run build` - 构建生产版本（使用 Turbopack）
- `npm start` - 启动生产服务器

### 开发服务器
开发服务器默认运行在 http://localhost:3000，支持热重载功能。

## 技术栈

- **框架**: Next.js 15.5.5 (App Router)
- **语言**: TypeScript 5.x
- **样式**: Tailwind CSS 4.x
- **字体**: Geist Sans & Geist Mono (通过 next/font)
- **构建工具**: Turbopack

## 项目架构

### 目录结构
```
├── app/                 # Next.js App Router 应用目录
│   ├── globals.css     # 全局样式和 Tailwind 配置
│   ├── layout.tsx      # 根布局组件
│   └── page.tsx        # 首页组件
├── public/             # 静态资源目录
├── next.config.ts      # Next.js 配置文件
├── tsconfig.json       # TypeScript 配置
└── package.json        # 项目依赖和脚本
```

### 关键配置特性
- **路径别名**: 使用 `@/*` 指向项目根目录
- **暗色主题**: 通过 CSS 变量支持明暗主题切换
- **字体优化**: 集成 Geist 字体族，支持 Sans 和 Mono 两种样式
- **样式系统**: 使用 Tailwind CSS v4 的 @theme 语法进行主题定制

### 布局系统
- 根布局 (`app/layout.tsx`) 定义了 HTML 结构和字体配置
- 全局样式 (`app/globals.css`) 包含 Tailwind 导入和 CSS 变量定义
- 支持明暗主题的自动切换

## 开发注意事项

- 项目使用 Turbopack 作为构建工具，开发体验更快速
- 所有 React 组件都应使用 TypeScript 编写
- 样式开发优先使用 Tailwind CSS 类名
- 遵循 Next.js App Router 的文件约定和最佳实践

---
> Source: [kongshine/omni-imaging](https://github.com/kongshine/omni-imaging) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
