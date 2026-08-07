## canvas-commerce

> - 本项目是 CanvasCommerce，一个计划开源、自托管的电商 AI 生图无限画布应用；对外正式发布前必须补充明确的开源许可证。

# 项目工作说明

## 项目概述

- 本项目是 CanvasCommerce，一个计划开源、自托管的电商 AI 生图无限画布应用；对外正式发布前必须补充明确的开源许可证。
- v1 默认用户自带 Provider API Key，默认 `AUTH_MODE=local`，不强制登录。
- 主要技术栈是 Next.js、React、TypeScript、Tailwind CSS、Radix/shadcn 风格组件、React Flow、Prisma、PostgreSQL、Redis、BullMQ。
- 开源 v1 当前只实现本地文件存储。`packages/storage` 保留对象存储抽象，但 OSS/S3/MinIO 驱动尚未实现，只能作为后续定制版或生产增强适配。
- v1 不默认使用 Nest。后续定制版可以新增 `apps/api` Nest 服务，但核心业务必须复用 packages。
- 优先遵循现有代码风格和目录边界，不要引入新的架构风格。

## 常用命令

- 安装依赖：`pnpm install`
- 本地开发：`pnpm dev` 或 `pnpm start`（热更新）
- Web 生产启动：`pnpm start:prod`（需要先 `pnpm build`；Worker 仍需独立常驻运行）
- Worker 开发：`pnpm dev:worker`
- 类型检查：`pnpm typecheck`
- 代码检查：`pnpm lint`
- 自动修复：`pnpm lint:fix`
- 单元测试：`pnpm test`
- 构建检查：`pnpm build`
- 格式化：`pnpm format`
- 提交引导：`pnpm commit`

## 目录结构

- `apps/web/`：Next.js 页面、轻量 API routes、SSE 入口和前端 UI。
- `apps/web/src/app/`：App Router 页面和 API route 入口。
- `apps/web/src/components/ui/`：基础 UI 组件，按 Stitch 设计定制。
- `apps/web/src/components/layout/`：顶栏、侧栏、页面壳等布局组件。
- `apps/web/src/components/canvas/`：画布相关展示组件。
- `apps/web/src/features/`：前端业务功能模块，如项目、画布、节点、资产、设置、Agent、生成、导出。
- `packages/shared/`：共享枚举、类型、schema 和前后端协议。
- `packages/core/`：可复用业务用例，API、worker、未来 Nest 服务都应调用这里。
- `packages/db/`：Prisma schema、migration 和数据库访问。
- `packages/storage/`：本地文件存储实现和对象存储扩展接口；开源 v1 尚未实现 S3/OSS/MinIO 驱动。
- `packages/provider/`：OpenAI、Anthropic、Google Gemini 及兼容协议的文本/图片调用，并包含 fal、Replicate、Vercel AI Gateway 等底层适配。
- `packages/queue/`：BullMQ 队列名、job payload、事件协议。
- `packages/worker/`：生图、导出、Agent 后台任务执行进程。
- `packages/agent/`：确定性 Skill、意图解析和节点铺图逻辑。
- `packages/config/`：环境变量和全局配置解析。
- `doc/`：产品、设计和任务文档；该目录不提交 Git。

## 编码规范

- UI 必须按 Stitch 当前采用屏幕实现，不要做成临时 demo 或通用后台样式。
- 代码必须重视可维护性、可读性和封装性；复杂业务流程、关键约束、非显而易见的实现原因需要写清楚注释。
- 前端样式必须分层：全局样式只放设计 token、reset、字体和通用变量；页面布局和组件样式放在对应页面、feature 或组件内。
- 前端目录必须按生产产品结构划分，页面、feature、layout、ui 基础组件、业务组件和 server 工具要边界清晰。
- 组件必须封装清楚，避免在页面文件里堆大段 UI 和业务交互；可复用 UI 进入 `components/ui/`，业务模块组件进入对应 `features/`。
- 页面语言以中文为主，CanvasCommerce、Agent、API Key、Provider、prompt 等专名可保留英文。
- Next.js route handler 只做 request/response 适配，不要堆核心业务逻辑。
- 核心业务逻辑优先放到 `packages/core/`。
- 跨端共享的类型、枚举和 schema 放到 `packages/shared/`。
- 默认情况下不要手动编写或修改 `migration.sql`；数据库迁移必须通过 Prisma schema 和 Prisma migrate 生成。
- 已经应用或提交的 migration 视为不可变文件，禁止修改、重命名或删除；修正数据库结构必须新增 migration。
- 正常开发只修改 `schema.prisma` 后运行 `pnpm db:migrate --name <change_name>`；随后检查生成 SQL，并运行 `pnpm db:generate`。
- `pnpm db:push` 只允许用于没有迁移历史、可随时丢弃的临时实验库，不得用于共享开发库、测试库或生产库。
- 生产和 CI 只使用 `pnpm db:deploy` 应用已有 migration，不得运行 `migrate dev`、`db push` 或 `migrate reset`。
- 发现 Prisma migration drift 时必须停止迁移；禁止手工修改 `_prisma_migrations`、checksum 或旧 migration 来掩盖差异。可丢弃的本地库也必须先获得明确确认才能 reset；不可丢弃环境需先备份，再做 baseline 或前向修复。
- migration 提交前至少运行 `pnpm db:status`，并在一套全新临时数据库上验证 `prisma migrate deploy` 可以从零建库。
- Provider 调用放到 `packages/provider/`，不要写进页面或 API route。
- 生图任务必须通过 `packages/queue/` 和 `packages/worker/` 执行，不能用普通 HTTP 请求同步长跑。
- 只有 Planner 产出的生图方案需要成本确认；用户确认方案即同时确认该方案中的 Provider 调用，不再增加第二次确认。
- 纯手动画布运行、已有节点重新生成，以及不经过 Planner 的 direct-edit 可以直接进入生图队列，无需成本确认。
- 文件存储必须通过 `packages/storage/`，不要在业务代码里直接写死本地路径或云厂商 SDK。
- 画布核心交互使用 React Flow，不手写无限画布底层能力。
- 图标优先使用 `lucide-react`。
- 通用弹窗、菜单、tooltip、popover 等交互优先使用 Radix/shadcn 风格组件并按 Stitch 改样式。
- 修改已有功能时，优先保持现有接口兼容。

## 禁止事项

- 不要提交 `.env`、token、密钥或真实 Provider API Key。
- 不要把 `doc/` 加回 Git 提交。
- 不要在 Planner 产出的生图方案未经用户确认时启动生成任务。
- 不要让文本节点自动把上游图片透传给下游图片生成节点。
- 不要把图片二进制存进数据库。
- 不要把业务逻辑写死在 React 组件或 Next.js route handler。
- 不要默认引入 Nest；只有用户明确要求定制版或后端拆分时再新增 `apps/api`。
- 不要升级核心依赖版本，除非用户明确要求。
- 不要删除用户已有改动。

## 验证要求

- 每次提交代码前必须人工审查完整 staged diff，确认行为边界、反例、测试覆盖、无关改动和敏感信息；自动测试通过不能替代代码审查。审查发现问题时必须先修复并重新验证，审查通过后才能提交。
- 修改 TypeScript 代码后，运行 `pnpm typecheck`。
- 修改 UI 或公共组件后，运行 `pnpm lint` 和相关构建检查。
- 修改工程配置、依赖或 monorepo 结构后，运行 `pnpm install`、`pnpm typecheck`、`pnpm lint`。
- 修改生成、队列、Provider、Storage 或 Agent 流程后，补充或更新相关测试。
- 如果测试或构建无法运行，在最终回复里说明原因和当前阻塞点。

## 常见坑

- 本项目使用 pnpm，不要使用 npm 或 yarn 安装依赖。
- Windows PowerShell 可能拦截 `pnpm.ps1`，可使用 `pnpm.cmd` 执行命令。
- `doc/` 已被 `.gitignore` 忽略，任务文档不会提交。
- 开源 v1 只能使用 `STORAGE_DRIVER=local`；不要把 MinIO/S3 当成必需服务，也不要在驱动实现完成前写成已经支持。
- 生成图是后台任务，必须依赖 Redis/BullMQ/worker。
- Vercel 适合部署 Web 预览，不是完整免费自托管方案。
- 修改配置文件后，需要重新启动开发服务器。

---
> Source: [yiqing168/canvas-commerce](https://github.com/yiqing168/canvas-commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
