## ai-coding-guide

> - 框架：（如 Next.js 14 / Spring Boot 3 / FastAPI）

# 项目名称

## 技术栈
- 框架：（如 Next.js 14 / Spring Boot 3 / FastAPI）
- 语言：（如 TypeScript strict / Java 17 / Python 3.12）
- 数据库：（如 PostgreSQL + Drizzle ORM）
- 测试：（如 Vitest / pytest / JUnit 5）

## 目录结构
- src/app/       — 页面和路由
- src/components/ — UI 组件
- src/services/   — 业务逻辑
- src/utils/      — 工具函数
- tests/          — 测试文件

## 常用命令
- 启动开发：pnpm dev
- 跑测试：pnpm test
- 类型检查：pnpm typecheck
- Lint：pnpm lint
- 构建：pnpm build

## 代码规范
- 使用 TypeScript strict mode，不允许 any
- 组件用 PascalCase，工具函数用 camelCase
- 每个文件单一职责，不超过 300 行
- 错误用自定义异常类，统一返回 { code, message, data } 格式

## 禁止事项
- 不要修改 src/legacy/ 下的代码
- 不要直接操作 DOM
- 不要手写 SQL，用 ORM
- 不要在代码里硬编码密钥或环境变量

---
> Source: [jnMetaCode/ai-coding-guide](https://github.com/jnMetaCode/ai-coding-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
