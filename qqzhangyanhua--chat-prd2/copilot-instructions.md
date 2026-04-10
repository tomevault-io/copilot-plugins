## chat-prd2

> 本仓库是 `pnpm` monorepo，核心应用只有两个：`apps/web` 为 Next.js 15 + React 19 前端，`apps/api` 为 FastAPI 后端。前端页面放在 `apps/web/src/app`，通用组件放在 `apps/web/src/components`，状态管理放在 `apps/web/src/store`，工具函数放在 `apps/web/src/lib`。后端按分层组织：`app/api` 路由、`app/services` 业务编排、`app/repositories` 数据访问、`app/schemas` 输入输出模型、`app/db` 持久化。测试分别位于 `apps/web/src/test` 和 `apps/api/tests`，设计与计划文档放在 `docs/superpowers/specs`、`docs/superpowers/plans`。

# Repository Guidelines

## 项目结构与模块组织
本仓库是 `pnpm` monorepo，核心应用只有两个：`apps/web` 为 Next.js 15 + React 19 前端，`apps/api` 为 FastAPI 后端。前端页面放在 `apps/web/src/app`，通用组件放在 `apps/web/src/components`，状态管理放在 `apps/web/src/store`，工具函数放在 `apps/web/src/lib`。后端按分层组织：`app/api` 路由、`app/services` 业务编排、`app/repositories` 数据访问、`app/schemas` 输入输出模型、`app/db` 持久化。测试分别位于 `apps/web/src/test` 和 `apps/api/tests`，设计与计划文档放在 `docs/superpowers/specs`、`docs/superpowers/plans`。

## 构建、测试与开发命令
先执行 `pnpm install` 安装工作区依赖。前端本地开发使用 `pnpm dev:web`，后端本地开发使用 `pnpm dev:api`。前端测试执行 `pnpm test:web`，生产构建校验执行 `pnpm --filter web build`。后端测试执行 `pnpm test:api`。涉及数据库结构变更时，在 `apps/api` 目录执行 `alembic upgrade head`。

## 编码风格与命名约定
不要顺手重排无关文件，优先延续现有风格。前端使用严格模式 TypeScript、2 空格缩进，并通过 `apps/web/tsconfig.json` 中的 `@/*` 别名引用源码。React 组件使用 `PascalCase`，函数和变量使用 `camelCase`，文件名保持 kebab-case，例如 `use-auth-guard.ts`。后端遵循 PEP 8、4 空格缩进、`snake_case` 模块命名，并尽量补全类型标注。仓库当前没有单独的 lint 脚本，因此提交前至少保证导入顺序、命名和周边文件一致。

## 测试指南
前端测试基于 Vitest，运行环境是 `jsdom`，公共初始化文件为 `apps/web/src/test/setup.ts`。组件测试命名为 `*.test.tsx`，纯逻辑或状态测试命名为 `*.test.ts`。后端测试使用 `pytest`，文件名遵循 `test_*.py`，例如 `test_auth.py`、`test_messages_service.py`。凡是修改行为、接口或状态流转，必须同步新增或更新至少一个自动化测试。

## 提交与合并请求规范
最近提交历史采用带作用域的 Conventional Commits，例如 `feat(web): ...`、`feat(api): ...`，文档和修复也应保持同一格式。每个提交只解决一个明确问题，避免把前后端无关修改混在一起。PR 至少应包含：变更摘要、关联 issue 或设计文档、测试结果；如果涉及界面改动，补充截图或录屏。

## 安全与配置提示
不要提交 `.env` 或 `apps/web/.env.local` 中的密钥、数据库地址和模型提供商凭据。修改认证、跨域、流式响应或 API 地址时，要同时检查前后端配置是否仍然匹配，避免只改一侧导致本地可用、联调失效。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qqzhangyanhua)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qqzhangyanhua)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
