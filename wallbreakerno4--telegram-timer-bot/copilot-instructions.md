## telegram-timer-bot

> 面向在本仓库中工作的智能编码代理：改动小、可验证、遵循既有模式；能写测试就先写测试。

# 仓库代理指南（telegram-timer-bot）
面向在本仓库中工作的智能编码代理：改动小、可验证、遵循既有模式；能写测试就先写测试。

## 项目做什么
- Cloudflare Workers 上的 Telegram Bot（Wrangler 部署）
- 私聊：`/start`、`/changetz` 通过 Inline Keyboard 选择并保存 IANA 时区
- 群聊：`/tz` 查自己或 reply 目标的当地时间；`/tza` 汇总本群已登记且“见过”的成员当地时间
- `/tzm`：把自然语言时间解析为单次时间点，并按成员时区展示（需要 Cloudflare `AI` binding）
- 存储：Cloudflare D1（表 `users` / `chat_users`）

## 项目概览与关键文件
- 语言/类型：TypeScript（`tsconfig.json`：`strict`、`isolatedModules`、`noEmit`）
- 测试：Vitest + `@cloudflare/vitest-pool-workers`（见 `vitest.config.mts`）
- 包管理器：pnpm（`pnpm-lock.yaml`）
- Worker 入口：`src/index.ts`（`wrangler.jsonc` -> `main`）
- D1 迁移：`migrations/0000_init.sql`（需与 `src/db.ts:initSchema` 同步）
- 生成类型：`worker-configuration.d.ts`（`wrangler types` 生成，勿手改）
- Wrangler 配置：`wrangler.jsonc`（包含 `nodejs_compat`；改 bindings/vars 会影响 typegen 与测试）

## 命令（构建 / 开发 / 测试）
权威来源：`package.json`。

### 安装
- `pnpm install`

### 本地开发
- `pnpm dev`（同 `pnpm start`）-> `wrangler dev`

### 部署
- `pnpm deploy` -> `wrangler deploy`

### 生成 Cloudflare 类型
- `pnpm cf-typegen` -> `wrangler types`
- 改了 `wrangler.jsonc` 的 bindings/vars 后，必须重跑 `pnpm cf-typegen`

### 测试（Vitest）
- 全量：`pnpm test`
- 单文件：`pnpm test -- test/db.spec.ts`（也可：`pnpm test -- --run test/db.spec.ts`）
- 单用例名：`pnpm test -- -t "returns 404 when token path is invalid"`
- 单用例 + 只跑一次：`pnpm test -- --run -t "..."`
- 常用：watch `pnpm test -- --watch`；只跑一次 `pnpm test -- --run`
  - 单文件 watch：`pnpm test -- --watch test/tz.spec.ts`

Workers pool：
- `vitest.config.mts` 通过 `defineWorkersConfig` 读取 `wrangler.jsonc`
- 测试使用 `cloudflare:test` 的 `env/createExecutionContext/waitOnExecutionContext`

### Lint / 格式化
- 目前没有 `lint`/`format` scripts（如果要加，写进 `package.json`，保持快速且可重复）
- 已存在配置：`.editorconfig`（tab、LF、去尾随空格）；`.prettierrc`（`printWidth=140`、单引号、分号、tab）
- 注意：`.editorconfig` 与 `.prettierrc` 的缩进（space vs tab）存在历史不一致；不要全仓扫格式化
- 新增或修改时：优先保持“同一文件内”风格一致；必要时以 `.prettierrc` 为目标（单引号/分号/printWidth）

### 类型检查（可选）
- 目前没有 `typecheck` script；需要时可直接跑：`pnpm exec tsc -p tsconfig.json`
- 说明：`tsconfig.json` 默认 `exclude: ["test"]`，`tsc` 不会检查测试文件；测试侧的类型问题主要靠 Vitest/编辑器提示

## Cursor / Copilot 规则
- 未发现：`.cursor/rules/`、`.cursorrules`、`.github/copilot-instructions.md`

## 常用工作流
- 启动本地：`pnpm dev`（Wrangler dev）
- 改动后最小验证：`pnpm test -- --run test/<相关文件>.spec.ts`
- 部署前：`pnpm test` 通过后再 `pnpm deploy`

## Git 与安全
- 不要提交敏感信息：真实 token、`.dev.vars*`、`.env*`（这些应只存在本地）
- 不要手改生成文件：`worker-configuration.d.ts`
- 提交信息：中文；尽量原子、聚焦
- 禁止用 `as any`、`@ts-ignore`、`@ts-expect-error` 去“压住”类型错误；应收窄类型或补齐分支

## 代码风格与约定（按现有代码来）

### 目录职责
- 路由/边界：`src/index.ts`
- DB 层：`src/db.ts`
- 纯工具：`src/time_format.ts`、`src/timezones.ts`
- callback 编解码：`src/callback_data.ts`
- Inline Keyboard 视图：`src/handlers/timezone_keyboard.ts`
- Telegram API 兼容封装：`src/telegram_api.ts`（不同版本 API 命名差异）
- 文本拼装与截断：`src/telegram_text.ts`（受 Telegram 4096 限制）

### import
- 第三方在前，本地相对路径在后；两组之间空一行
- 适用时用 type-only import：`import { type Foo } from '...'`

### 命名
- 文件：`snake_case.ts`
- 变量/函数：`camelCase`
- 类型/接口：`PascalCase`
- 常量：`SCREAMING_SNAKE_CASE`

### 类型与返回值
- 避免 `any`；优先窄类型与显式返回值（`Promise<...>`、`T | null`）
- “可预期失败”用 result union：`{ ok: true; value: T } | { ok: false; error: ... }`（见 `src/callback_data.ts`、`src/time_format.ts`）
- 用 `as const`/`readonly` 固化字面量与不可变数据；用 `satisfies` 做结构约束但不扩大类型（见 `src/index.ts`）

### 错误处理
- 校验/解析类错误：返回结构化错误（code/message）而不是随意 throw
- 只有真正异常才 throw；在边界 catch 并转成用户可理解的提示（例如 Telegram callback 流）
- 不要吞错：catch 后要么返回安全值、要么映射为结构化错误、要么给用户反馈
- 允许在用户交互边界做“兜底吞错”（比如 callback 流只提示重试），但不要悄悄失败；必要时在边界 `console.error`

### 异步与副作用
- 以 `async/await` 为主；handler 统一返回 `Promise<Response>`
- 外部 I/O（DB、fetch Telegram API）集中在边界处，纯逻辑尽量写成纯函数

### 数据库（D1）
- 使用 prepared statement：`env.DB.prepare(sql).bind(...).run()/first()/all()`
- schema 要和 `migrations/*.sql` 同步
- 已知坑：workers pool 下 `env.DB.exec()`（尤其多语句）可能不稳定；优先每条语句 `prepare(...).run()`
- 新增表/字段时：先加 `migrations/xxxx_*.sql`，再同步更新 `src/db.ts:initSchema`

### Cloudflare env / secrets
- 必需 secret：`SECRET_TELEGRAM_API_TOKEN`（声明在 `src/env.d.ts`）
- D1 binding：`DB`（见 `wrangler.jsonc` 与 `worker-configuration.d.ts`）
- AI binding：`AI`（用于 `/tzm`，见 `wrangler.jsonc`）
- 不要提交真实 token；生产用 Wrangler secrets
- 本地敏感信息放 `.dev.vars*` / `.env*`（仓库已在 `.gitignore` 中忽略，保留 `*.example`）

新增 env/binding 的改动清单：
- `src/env.d.ts` 增补类型 -> `wrangler.jsonc` 增补 vars/bindings -> `pnpm cf-typegen`

### Worker 路由契约（对外接口）
- webhook：`POST /{SECRET_TELEGRAM_API_TOKEN}`；token 路径不匹配返回 `404`
- 设置 webhook：`GET /{token}?command=set`（内部调用 Telegram `setWebhook`）

## 测试约定
- Vitest：`describe/it/expect`
- Workers：`cloudflare:test` 的 `createExecutionContext()` + `waitOnExecutionContext(ctx)` + `env`
- mock 外部请求：`vi.stubGlobal('fetch', ...)`；并在 `afterEach` 清理 `vi.unstubAllGlobals()`、`vi.restoreAllMocks()`
- 断言 Telegram API 调用时，优先解析 URLSearchParams/JSON 参数，不做脆弱的整串 URL 匹配
- 时间相关逻辑：`vi.useFakeTimers()` + `vi.setSystemTime(...)`，并在 `afterEach` 还原

常见断言策略：
- 对 outbound `fetch`：取 `Request.url` 再用 `new URL(...)` 分析 path/query
- body 可能是 JSON 或 form：优先 `JSON.parse`，失败再用 `URLSearchParams`

## 新增功能检查清单
- 新增/修改对外行为：补或改对应的 `test/*.spec.ts`
- 需要新 env/binding：同步 `src/env.d.ts` + `wrangler.jsonc` + `pnpm cf-typegen`
- 需要改 DB schema：先加 `migrations/xxxx_*.sql`，再同步 `src/db.ts:initSchema`
- 引入新依赖：优先确认 Workers 运行时可用；更新 `pnpm-lock.yaml`

## 变更卫生
- diff 保持聚焦；修 bug 不要顺手大重构
- 行为改动必须补测试（`test/*.spec.ts`）
- 改 bindings/config 时，同步更新 typegen/文档/测试
- git commit message：中文

## 小贴士
- Telegram 文本长度限制：实现里用到 `4096`（见 `src/index.ts`）
- callback_data 体积限制：编码器有 64 bytes 上限（见 `src/callback_data.ts`）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WallBreakerNO4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/WallBreakerNO4)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
