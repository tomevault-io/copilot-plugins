## app-architecture

> PromptGenie 是一个基于 Tauri 的桌面应用，用于管理 AI 提示词模板。

# PromptGenie 应用架构

PromptGenie 是一个基于 Tauri 的桌面应用，用于管理 AI 提示词模板。

## 技术栈
- 前端: React + TypeScript + Tailwindcss
- 后端: Rust (Tauri)
- 数据库: SQLite (通过 tauri-plugin-sql)
- 前端使用pnpm作为包管理
- Icon使用lucide icons

## 一些重要的技术细节
- 我用的是tauri v2，你一定要注意这点，你知道的很多技术细节和方案可能已经过时了！
- 所有的数据库id务必使用uuid v7，不要用v4
- 能在前端做的事情尽量放在前端代码完成，不得已再考虑在rust端实现。

## 关键文件
- Rust 后端: [src-tauri/src/lib.rs](mdc:src-tauri/src/lib.rs) - Tauri 应用入口及数据库初始化
- 数据库模式: [src-tauri/db/schema.sql](mdc:src-tauri/db/schema.sql) - 包含数据库表定义
- 数据库服务: [src/services/db.ts](mdc:src/services/db.ts) - 前端数据库访问服务

## 数据流程
1. 应用启动时在 Rust 后端初始化 SQLite 数据库
2. 前端通过 tauri-plugin-sql 与数据库交互
3. 数据库服务层处理所有 CRUD 操作
4. React 组件消费和显示数据

---
> Source: [ChrisZou/promptgenie](https://github.com/ChrisZou/promptgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
