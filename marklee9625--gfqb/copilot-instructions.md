## gfqb

> 工法情报编辑器：React 19 + TypeScript + Vite + IndexedDB 的本地优先情报编辑工具，BFF 代理接 DeepSeek API。**始终使用简体中文回复。**

﻿# AGENTS.md

## 项目概述

工法情报编辑器：React 19 + TypeScript + Vite + IndexedDB 的本地优先情报编辑工具，BFF 代理接 DeepSeek API。**始终使用简体中文回复。**

## 可用技能（`.codex/skills/`）

| 技能 | 用途 |
|------|------|
| add-article | 新建文章 |
| ai-meta | AI 元数据/图谱/选题 |
| check-app | 全面检查 |
| app-audit | 代码分析 |
| db-inspect | IndexedDB 检查 |
| export-data | 数据导出 |
| git-upload | 提交推送 |

## 关键命令

```powershell
npm run dev:all      # 并行启动 Vite :4512 + BFF :4513（开发推荐）
npm run dev          # 仅 Vite :4512
npm run server       # 仅 BFF :4513
npm run build        # 生产构建 → dist/
npm run test         # vitest run
npm run typecheck    # tsc --noEmit
start.ps1            # Windows 一键启动（检查依赖/清理残留进程）
```

**开发环境必须启动 BFF**（Vite 的 `/api` 代理转发到 :4513）。

## 环境变量（.env.local）

```env
DEEPSEEK_API_KEY=sk-xxx           # BFF 后端专用，绝不能加 VITE_ 前缀
PROXY_SECRET=sws-gongfa-proxy-xxx # BFF 鉴权密钥
```

## 架构要点

- App.tsx 是状态中枢（唯一 state 集中地）→ Hooks → Services → IndexedDB
- 数据落在 IndexedDB（db.ts 单例）

> 完整技术栈、代码规范见同目录 `CLAUDE.md`。

---
> Source: [MarkLee9625/GFQB](https://github.com/MarkLee9625/GFQB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
