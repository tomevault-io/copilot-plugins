## edgeever

> 本文件用于约束和指导参与本项目的 AI 代理与协作者。除非用户明确给出更高优先级的指令，否则应遵守以下规则。

# AGENTS.md

本文件用于约束和指导参与本项目的 AI 代理与协作者。除非用户明确给出更高优先级的指令，否则应遵守以下规则。

## 项目背景与技术栈

涉及本项目的背景、定位、部署信息与技术栈说明时，请优先参考 `README.md`。

## Git 分支约束

严禁创建新的 Git 分支；所有修改、提交和推送都必须直接在 `main` 分支上完成。

## Cloudflare 自动部署约束

当用户要求“根据 GitHub 项目链接把本项目安装部署到 Cloudflare”时，应优先按 `docs/agent-deploy-cloudflare.md` 执行。

推荐流程：

```sh
bun install
EDGE_EVER_PASSWORD='<首次登录密码>' bun run deploy:setup
bun run deploy:doctor
bun run deploy
```

如果用户没有提供首次登录密码，应只询问这一个必要信息，或在用户同意后生成随机密码。Cloudflare 授权、账号、D1/R2 资源、Worker 名称、自定义域名等私有配置必须来自用户环境、Cloudflare MCP/插件、Wrangler 登录态或 `.env.local`，严禁硬编码到仓库文件。

部署脚本必须通过 `scripts/run-wrangler.mjs` 读取 `.env.local` 并生成临时 Wrangler 配置。不要直接修改 `wrangler.toml` 来写入个人 `database_id`、bucket 名称、Worker 名称或 route。

## 本地启动约束

本地预览或调试时，必须优先使用 `bun run dev` 启动完整开发环境，让 API 通过 `scripts/run-wrangler.mjs` 读取 `.env.local` 中的个性化实例配置。实例名称、D1/R2 资源、账号等本机私有配置均以 `.env.local` 为准，严禁在代理指令或代码中硬编码个人实例名。

除非用户明确要求只启动前端静态界面，否则不要单独运行 `bun run dev:web`；该命令不会启动 API，也不会保证读取 `.env.local` 中的实例配置，容易导致前端请求 `127.0.0.1:8787` 失败或误判环境。

## 组件复用与造轮子约束

UI 功能应尽量复用 `shadcn/ui` 等现有 UI 组件。在实现其他功能时，也应优先采用成熟、稳定的开源组件或库，绝对禁止在没有充分必要性的前提下自行从零造轮子。

为方便代码维护，当页面或功能模块出现复杂结构、重复布局或潜在复用场景时，应视情况封装为独立组件，保持页面入口聚焦于组合与数据传递。

---
> Source: [tianma-if/edgeever](https://github.com/tianma-if/edgeever) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
