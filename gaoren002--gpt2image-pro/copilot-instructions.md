## gpt2image-pro

> > 镜像文件：`CLAUDE.md` 与 `AGENTS.md` 必须逐字一致（CI `docs-mirror` 强制；改一个须同步另一个）。

# CLAUDE.md / AGENTS.md

> 镜像文件：`CLAUDE.md` 与 `AGENTS.md` 必须逐字一致（CI `docs-mirror` 强制；改一个须同步另一个）。
> 本文件是所有协作 Agent 的共享行为准则与项目向导。持久记忆在 `./docs/`，不要依赖会话上下文。

## 项目

GPT2Image-Pro：AI 图像生成平台。Turborepo monorepo（pnpm + Next.js 16 App Router + React 19 + TypeScript strict）。

```
apps/web/          主应用与管理后台（src/app 路由组、src/features 业务模块）
packages/database/ Drizzle ORM schema 与 DB 连接（@repo/database、@repo/database/schema）
packages/shared/   业务逻辑 auth/credits/storage/payment/subscription...（@repo/shared/<module>）
packages/ui/       Shadcn/UI 组件（@repo/ui/components/<name>）
```

技术栈：Drizzle ORM + PostgreSQL、Better Auth、Zod + next-safe-action、Creem 支付、next-intl、Fumadocs MDX、Biome、Vitest。
部署：Docker Compose + Nginx + Certbot（DigitalOcean）。CI/CD 详见 `docs/CI-CD.md`。

## 核心约束（MUST）

硬约束，优先级高于一切默认行为。所有产出须 **可追溯、可验证、可解释**。

**语言与排版**
- 对话回复用简体中文；代码标识符用英文；注释用简体中文。
- 永不使用 emoji。回复、代码、注释、提交信息、文档及任何写入文件，一律不用。

**代码风格**
- TypeScript `strict`；禁止 `any`（`noExplicitAny: error`），必要时用 `unknown` 加类型收窄。
- 写"读起来像周围代码"的代码：匹配既有命名、缩进、惯用法与文件组织，不擅自引入新风格。
- 命名见名知意；函数小而专注，文件单一职责；文件过大即是拆分信号。
- 不留死代码、被注释掉的代码、墓碑注释或 TODO 假完成；删除即彻底删除。
- 格式由 Biome 统一（双引号、分号、2 空格、行宽 80）；提交前 `biome lint` 无 error（告警级如 `noNonNullAssertion` 不阻断）。
- Server Components 优先，`'use client'` 仅在必要处；跨包用 `@repo/*`，包内用 `@/*`；i18n 导航从 `@/i18n/routing` 取。

**注释（人类与 LLM 皆可轻易理解）**
- 每个文件：文件级注释说明职责、使用方、关键依赖。
- 每个函数/组件：用途、参数、返回、副作用、边界与失败模式。
- 复杂逻辑（并发、事务、幂等、金额/积分、安全校验）：解释为何这样写（WHY），而非仅做了什么。
- 注释与代码同步；过期注释视为 Bug。

**Git 与版本**
- 持续小步提交，每完成一部分即 commit；关键节点打 tag。Conventional Commits（`type(scope): 摘要`，正文说明 WHY）。
- 版本格式 `v<MAJOR>.<MINOR>.<PATCH>-<alpha|beta|rc>.<N>`（正式版去后缀）。
- 分支：单分支 `main`（默认 / 开发 / 生产），开发即提交并推送 `main`；已无 `dev`。
- 不用 `--no-verify` 绕过校验；force-push、`reset --hard`、删分支/tag 等破坏性操作先确认。

**健壮性**
- 校验一切外部输入（用户、API、Webhook、DB 结果、第三方响应，优先 Zod）；不信任的数据视为可能非法。
- 不吞异常：要么处理，要么显式上抛并记日志（Pino）；面向用户的错误友好可定位。
- 金额/积分/扣费/配额须考虑幂等与并发，优先用数据库约束兜底，杜绝双花/少扣/多扣。
- 可选服务（Redis、Axiom、Sentry）未配置时优雅降级。提交前过质量门：`turbo typecheck`、`turbo lint`、`turbo test` 全绿。

**测试**
- 核心逻辑（积分、扣费、订阅、鉴权、幂等、API）必须有单测；覆盖正常、边界、失败、并发/重复请求。
- 修 Bug 先写复现测试再修。`packages/shared`、`apps/web` 的 vitest 为 DB-free；纯函数须抽到不 import `@repo/database` 的模块才能单测。
- 不用 skip、注释断言、弱化断言制造假绿灯。

**安全**
- 机密（密钥、令牌、口令）只存 `.env.local` 与部署环境，严禁提交或写入文档/日志/注释；发现泄露立即提示轮换。
- 防 SQL 注入、XSS、CSRF、SSRF、命令注入、路径穿越、越权（IDOR）；服务端动作走 `protectedAction`/`adminAction`，每处资源访问校验归属与权限。
- 向上游/第三方转发请求不得携带客户端凭据（`Authorization`/Cookie）；仅对第一方（same-origin）转发。
- 依赖升级先评估来源与破坏性；遵循"纯中转/不记录"等隐私边界，不在该路径落库或留存用户数据。

**工作方式**
- 不确定即查，禁止猜测：用工具与可靠资料取证（读代码、查文档、跑验证），不靠直觉作答或写码。
- 独立任务用子 Agent 承接以隔离上下文；并行的重复任务用子 Agent 并行。
- 持久化记忆写入 `./docs/`：`MEMORY.md`（索引）、`memory/`（详情）、`plan/`（计划）、`TODO.md`（待办，定期清理）。

## Agent 集成架构约束（统一接口层优先）

后续开发须遵循"功能先暴露为接口、再对接 agent"的分层架构，详见 `docs/plan/2026-05-31-agent-integration-architecture.md`（主设计）与 `docs/plan/2026-05-31-feature-interface-inventory.md`（全功能接口盘点表）。

- 开发任何新功能或改造现有功能，必须先在【统一接口层 UOL / Operation Registry】（`packages/shared/src/uol/`）暴露为一个 `defineOperation()` 注册项：携带 Zod 输入 schema、明确输出类型、声明式权限（AccessRequirement）、能力位（复用 `plan-capabilities`）、只读/破坏性标志、副作用与幂等声明、传输无关的 `execute(input, principal, ctx)`。功能必须先成为"可被 agent 调用的接口"，再按需经其他传输暴露。
- 传输层只做薄适配：server-action / api-route / cron / webhook 退化为"解析请求 → 构造 Principal → `invokeOperation(name, input, principal)` → 编码响应"。鉴权、能力校验、审计、幂等、错误映射单点在 `invokeOperation` 网关完成，不在各传输层重复。
- MCP 对接层是接口层之上的可配置适配器（默认关闭，`MCP_ENABLED`；单一管理秘钥 Bearer 恒定时间比对鉴权），面向任意外部 agent。MCP 不直接调 service-fn，只调 registry operation。
- 站内内置 Agent 直连接口层（进程内直调，不经 MCP），与 MCP 共享同一 registry、Principal、权限模型、幂等与审计装饰；内置 Agent 须与 Next.js 同进程运行。
- 不可破坏的不变量：单一图像管线 `runImageGenerationForUser`；财务真相在 `credits_transaction`（双重记账）；扣费幂等键 `(user_id, type, source_ref)`，发放/退款幂等键 `credits_batch(source_type, source_ref)`；能力判定唯一来源 `plan-capabilities.ts`；底层 service 自带 `db.transaction` 不可嵌套外层事务。新写的扣费/发放/工单创建等操作必须带幂等键（sourceRef/clientRequestId）。
- 涉及财务（credits）、存储（storage）、内容审核（moderation fail-closed 透传）的接口化最后做，且必须先补幂等键与单测。

## 关键事实（改动前必读）

- **单一图像管线**：5 个 v1 handler（`apps/web/src/features/external-api/handlers/`）最终汇入 `image-generation/operations.ts` 的 `runImageGenerationForUser`；单点改造覆盖全部 v1 路径。
- **财务真相在 `credits_transaction`（双重记账），不在 `generation` 行**（后者仅历史/画廊展示）。
- **幂等键**：扣费 `consumeCredits(sourceRef)` 加 `credits_transaction (user_id, type, source_ref)` per-user 偏唯一索引（迁移 0029，按用户分桶防跨用户误命中）；发放/退款 `credits_batch (source_type, source_ref)`。
- **迁移手写幂等 SQL**（`packages/database/drizzle/NNNN_*.sql` 加手动登记 `meta/_journal.json`），不用 `drizzle-kit generate`（快照漂移会进交互模式）。
- **套餐能力矩阵**：`plan-capabilities.ts`；新增能力位须同步 `system-settings/definitions.ts` 示例与 `system-settings-panel.tsx`，否则同步测试失败。
- **Server Action 分层**（`@repo/shared/safe-action.ts`）：`actionClient` 基础、`protectedAction` 带 `ctx.userId`、`adminAction` 要管理员。

## 命令与环境

```bash
turbo dev | build | typecheck | lint | test        # monorepo 根
pnpm --filter @repo/web dev                         # 主应用 (port 3000)
pnpm --filter @repo/database db:push | db:studio    # 数据库
docker compose build | up -d | logs -f web          # Docker
```

必需 env（见 `.env.example`）：`DATABASE_URL`、`BETTER_AUTH_SECRET`、`BETTER_AUTH_URL`。默认图像后端在后端池配置，而非环境变量。

---
> Source: [gaoren002/GPT2Image-Pro](https://github.com/gaoren002/GPT2Image-Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
