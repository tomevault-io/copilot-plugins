## savepoint

> 本仓库是基于 Next.js App Router 的 TypeScript 项目，使用 React 19、Tailwind CSS 4、PostgreSQL 和 `pnpm`。页面与路由位于 `src/app`，主页面是 `src/app/page.tsx`，全局布局是 `src/app/layout.tsx`，全局样式在 `src/app/globals.css`。后端接口使用 Route Handlers，统一放在 `src/app/api`。可复用 UI 组件位于 `src/components`，客户端请求封装位于 `src/lib`，自定义 Hook 位于 `src/hooks`。服务端代码集中在 `src/server`，其中数据库结构检查在 `src/server/db`，鉴权在 `src/server/auth`，短信在 `src/server/sms`，实时语音识别签名在 `src/server/asr`，上传在 `src/server/uploads`，创作记录逻辑在 `src/server/conversations`。静态资源放在 `public`，引用示例：`/pixel_camping.png`。

# Repository Guidelines

## 项目结构与模块组织

本仓库是基于 Next.js App Router 的 TypeScript 项目，使用 React 19、Tailwind CSS 4、PostgreSQL 和 `pnpm`。页面与路由位于 `src/app`，主页面是 `src/app/page.tsx`，全局布局是 `src/app/layout.tsx`，全局样式在 `src/app/globals.css`。后端接口使用 Route Handlers，统一放在 `src/app/api`。可复用 UI 组件位于 `src/components`，客户端请求封装位于 `src/lib`，自定义 Hook 位于 `src/hooks`。服务端代码集中在 `src/server`，其中数据库结构检查在 `src/server/db`，鉴权在 `src/server/auth`，短信在 `src/server/sms`，实时语音识别签名在 `src/server/asr`，上传在 `src/server/uploads`，创作记录逻辑在 `src/server/conversations`。静态资源放在 `public`，引用示例：`/pixel_camping.png`。

## 构建、测试与开发命令

首次开发前运行 `pnpm install`。常用命令如下：

- `pnpm dev`：先检查并修复 PostgreSQL 表结构，再启动本地 Next.js 开发服务。
- `pnpm build`：执行生产构建，并进行 Next.js 类型和构建校验。
- `pnpm start`：生产环境启动命令，启动前同样执行数据库结构检查。
- `pnpm db:check`：按 `docs/database-schema.md` 检查并自动修复数据库结构。
- `pnpm lint`：使用 `eslint.config.mjs` 运行 ESLint。

## 代码风格与命名规范

优先编写 TypeScript 和 React 函数组件。代码使用两个空格缩进、双引号和分号。组件文件使用 PascalCase，例如 `BookChat.tsx`；Hook 文件使用 camelCase 且以 `use` 开头，例如 `useAudio.ts`。跨目录导入优先使用 `@/*` 别名。类型只在多个模块复用时抽离，否则保留在拥有该逻辑的文件附近。

## 测试指南

当前项目尚未配置测试框架，也没有 `test` 脚本。提交前至少运行 `pnpm lint` 和 `pnpm build`。涉及数据库结构变更时，还需要在可用 PostgreSQL 环境中运行 `pnpm db:check`。后续如添加测试，建议将小型组件测试放在源码附近，例如 `BookChat.test.tsx`，并同步更新 `package.json` 脚本。

## 提交与 Pull Request 规范

现有 Git 历史使用简短中文提交信息，例如 `项目初始化`、`功能：集成腾讯云，实现短信认证`。新提交应保持单一主题，避免混入无关改动。Pull Request 需要包含变更摘要、用户可见影响、已执行的验证命令；涉及 UI 改动时，应附截图或录屏；有关联 issue 时应在描述中链接。

## 安全与配置说明

本地开发请从 `.env.example` 复制 `.env.local`，并至少配置 `DATABASE_URL`。语音输入使用腾讯云实时语音识别，相关环境变量为 `ASR_APP_ID`、`ASR_SECRET_ID`、`ASR_SECRET_KEY`，只允许在服务端签名接口中读取。不要提交真实的数据库、短信、语音识别、会话或 AI API 密钥。不要提交 `node_modules`、构建产物、生成目录，或 `storage` 下的本地上传文件。

---
> Source: [Wanyi424/SavePoint](https://github.com/Wanyi424/SavePoint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
