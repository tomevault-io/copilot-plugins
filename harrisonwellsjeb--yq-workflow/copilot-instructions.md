## yq-workflow

> - **pnpm workspace**（`pnpm@9.7.0`），Node ≥20，全包 ESM（`"type": "module"`）

# yq-workflow — AGENTS.md

请始终用中文回答。

## Monorepo 结构

- **pnpm workspace**（`pnpm@9.7.0`），Node ≥20，全包 ESM（`"type": "module"`）
- **packages/**（API 客户端库，无构建步骤，被 esbuild 直接引用源码）：
  - `teambition` — Teambition Open API 客户端（任务/迭代/自定义字段/RTF 渲染/测试用例）
  - `gitlab` — GitLab Open API 客户端（MR/讨论/用户/合并成员，含持久缓存）
  - `figma` — Figma Open API 客户端（节点树 → Markdown 文档提取，支持 EASY-DOCS 与文档容器两种模式）
  - `dingtalk` — 钉钉文档 API 客户端（异步导出 Markdown：提交 → 轮询 → 下载）
- **apps/**（应用层，各自 esbuild 打包）：
  - `yunqudev` — opencode 研发 agent 插件（plugin.ts + skills + scripts + utils），依赖全部 4 个 API 包
  - `webhook` — Fastify webhook 服务（GitLab MR 合并 → Teambition 状态联动 + 可合并/review note 邮件通知），依赖 teambition + gitlab
  - `teambition-oauth-server` — 四平台 OAuth token 兑换/刷新服务（Teambition + GitLab + Figma + 钉钉），plugin 免登前置
  - `e2e-platform` — E2E 测试管理平台（Fastify + SQLite + Midscene.js + Puppeteer + React 19 前端）

## 关键命令

| 命令             | 含义                                                    |
| ---------------- | ------------------------------------------------------- |
| `pnpm build`     | 构建 yunqudev + webhook（esbuild 打包）                 |
| `pnpm typecheck` | `pnpm -r exec tsc --noEmit`（全包检查）                 |
| `pnpm test`      | `pnpm -r test`（全包测试）                              |
| `pnpm lint`      | ESLint 扁平配置（`eslint.config.js`），忽略 `dist/`     |
| `pnpm format`    | Prettier（singleQuote, noSemi, printWidth 100）         |
| `pnpm verify`    | 构建 + 类型检查 + 测试 + lint + 格式化 — 发版前必须执行 |

各包测试命令（作用域不同）：

- **yunqudev**：`pnpm --filter @yq-workflow/yunqudev test` — 运行 `src/utils/*.test.ts`
- **webhook**：`pnpm --filter @yq-workflow/webhook test` — 运行 `test/*.test.ts`
- **gitlab**：`pnpm --filter @yq-workflow/gitlab test` — 运行 `src/*.test.ts`
- **figma**：`pnpm --filter @yq-workflow/figma test` — 运行 `src/*.test.ts`
- **dingtalk**：`pnpm --filter @yq-workflow/dingtalk test` — 运行 `src/*.test.ts`
- **e2e-platform**：`pnpm --filter @yq-workflow/e2e-platform test` — 运行 `test/*.test.ts`

## 构建注意事项

- **yunqudev 与 webhook 的 `dist/` 均不入库**（已 gitignore）。yunqudev 产物通过 `pnpm publish:plugin` 发布到独立插件仓库 `yunqudev-plugin.git`（用户安装源），源码仓库只保留 `.ts` 源码。
- **禁止手改 `dist/`** — 始终使用 `pnpm build` 重新生成。
- yunqudev 构建：esbuild 多入口从 `src/{scripts,utils}/` 打包到 `dist/scripts/`（内联 `@yq-workflow` 包与 turndown/linkedom，`rtfToMarkdown.ts` 单独打包）；`--target opencode` 额外打包 `opencode/plugin.ts` → `dist/opencode/plugin.js`，并拷贝 `scripts/`、`assets/`、`skills/` 到 `dist/opencode/`。
- Webhook 构建：esbuild 单文件打包 `dist/server.bundle.js`，带 `createRequire` banner（解决 fastify CJS 传递依赖）。
- e2e-platform 构建：分前后端——server 用 esbuild 打包，web 用 Vite 构建到 `web/` 目录由 server 静态托管。
- 完整插件健康检查：`pnpm --filter @yq-workflow/yunqudev verify`（构建 + 类型检查 + 测试）。
- 发布到用户：`pnpm publish:plugin`（根目录）— 先 build，再把 `dist/opencode/` rsync 推到 `yunqudev-plugin.git` 并打版本 tag。

## 凭证配置

凭证分两条路径（不同 app 用不同机制）：

- **plugin（yunqudev）— OAuth 免登**：用户无需手写任何凭证文件。broker 地址与应用 ID 已硬编码进插件脚本。首次使用时调 `yunqudev_login` 工具（参数 `platform: teambition|gitlab|figma|dingtalk`），自动打开浏览器完成 OAuth，写入 `~/.yq-workflow/user-token.json`（token 自动刷新）。
- **webhook / oauth-server — 手动配置**：用 `~/.yq-workflow/secrets.json`（开发机共享）或环境变量 `YQ_WORKFLOW_CONFIG=<路径>`（生产，兼容旧版 `GW_CONFIG`）。webhook 需要 `teambition.{APP_ID,APP_SECRET,ORG_ID,OPERATOR_ID}` + `gitlab.{TOKEN,API_BASE,webhookSecret}`；oauth-server 需要 `TB_APP_ID/TB_APP_SECRET/GITLAB_APP_ID/GITLAB_APP_SECRET/FIGMA_APP_ID/FIGMA_APP_SECRET/DINGTALK_APP_KEY/DINGTALK_APP_SECRET` 等环境变量（详见各 app README）。
- **本地覆盖**（已 gitignore）：`**/secrets.local.json`、`**/config.local.json`（仅 webhook 路径有效）。

## 代码规范

- 所有源码为 `.ts`（无 `.js`/`.mjs` 源码文件）。函数签名需写 JSDoc。
- ESLint + Prettier 必须在提交前通过 — 执行 `pnpm verify` 统一检查。
- packages 无运行时依赖（figma 除外，依赖 `@figma/rest-api-spec` 类型）；所有运行时依赖（`turndown`、`linkedom`、`fastify`、`better-sqlite3`、`puppeteer`、`react` 等）仅在打包它们的 apps 中。
- 无 CI、无 GitHub Actions、无 pre-commit hooks。

## 模块详解

完整架构与各模块设计说明见 [docs/architecture.md](./docs/architecture.md)。

---
> Source: [HarrisonWellsjeb/yq-workflow](https://github.com/HarrisonWellsjeb/yq-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
