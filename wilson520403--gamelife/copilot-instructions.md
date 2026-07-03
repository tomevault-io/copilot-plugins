## gamelife

> `src/app` 存放 Next.js App Router 页面、布局、加载态和 API 路由。通用组件在 `src/components`，其中 `dashboard`、`library`、`calendar`、`review` 等目录按功能拆分，基础 UI 组件位于 `src/components/ui`。业务逻辑与平台同步代码集中在 `src/lib`，自定义 Hook 在 `src/hooks`，共享类型在 `src/types`。数据库相关文件位于 `prisma` 和 `supabase/migrations`，静态资源在 `public`，Playnite 扩展位于 `extensions/playnite`。

# Repository Guidelines

## 项目结构与模块组织
`src/app` 存放 Next.js App Router 页面、布局、加载态和 API 路由。通用组件在 `src/components`，其中 `dashboard`、`library`、`calendar`、`review` 等目录按功能拆分，基础 UI 组件位于 `src/components/ui`。业务逻辑与平台同步代码集中在 `src/lib`，自定义 Hook 在 `src/hooks`，共享类型在 `src/types`。数据库相关文件位于 `prisma` 和 `supabase/migrations`，静态资源在 `public`，Playnite 扩展位于 `extensions/playnite`。

## 构建、测试与开发命令
仓库使用 `pnpm-lock.yaml`，默认请使用 `pnpm`。

- `pnpm dev`：启动本地开发服务。
- `pnpm build`：生成生产构建，同时检查集成层面的错误。
- `pnpm lint`：运行 ESLint（Next.js + TypeScript 规则）。
- `pnpm db:push`：将 Prisma Schema 推送到当前数据库。
- `pnpm db:seed`：执行 `prisma/seed.ts` 初始化数据。
- `pnpm sync:playnite`：运行远程 Playnite 同步脚本。
- `pnpm sync:playnite-local`：运行本地直连的 Playnite 同步脚本。
- `pnpm get-xbox-token`：启动 Xbox Token 获取流程。

## 代码风格与命名约定
项目启用 TypeScript `strict` 模式；模块边界处保持类型明确，优先使用 `@/*` 别名导入。遵循现有风格：2 空格缩进、保留分号、使用双引号。React 组件导出使用 PascalCase，自定义 Hook 使用 `useXxx`，文件名使用 kebab-case，例如 `summary-cards.tsx`、`sync-playnite.mjs`。App Router 文件保持约定命名，如 `page.tsx`、`loading.tsx`、`route.ts`。

## 测试要求
当前仓库没有正式的自动化测试框架，也没有覆盖率门禁。提交前至少运行 `pnpm lint` 和 `pnpm build`。如果改动涉及同步逻辑、脚本或数据处理，请直接运行对应脚本验证，例如 `npx tsx scripts/test-xbox.ts`，并在 PR 中写明手动验证步骤与结果。

## 提交与 Pull Request 规范
提交信息沿用现有 Conventional Commit 风格，例如 `feat:`、`fix:`、`refactor:`、`chore:`，后接简短的祈使句说明。每次提交只解决一个清晰问题。PR 需包含变更说明；如有关联 Issue，请附链接；涉及界面改动请附截图；涉及环境变量、数据库结构或迁移时请明确标注影响范围。

## 安全与配置提示
从 `.env.example` 复制本地环境配置，不要提交真实密钥。`SUPABASE_SERVICE_ROLE_KEY`、各平台 Token 以及 `CRON_SECRET` 都应视为仅服务端可用的敏感信息。任何影响同步鉴权、定时任务或外部平台接入的改动，都需要在 PR 中单独说明。

---
> Source: [Wilson520403/GameLife](https://github.com/Wilson520403/GameLife) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
