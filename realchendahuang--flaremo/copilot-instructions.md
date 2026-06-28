## flaremo

> 这份文件给 Codex、Claude Code、Cursor Agent 等自动化编码工具使用。目标是让 Agent 能稳定理解、修改、验证和部署 FlareMo。

# FlareMo Agent 指南

这份文件给 Codex、Claude Code、Cursor Agent 等自动化编码工具使用。目标是让 Agent 能稳定理解、修改、验证和部署 FlareMo。

## 项目定位

FlareMo 是一个 Cloudflare 原生的个人知识管理系统：

- 前端和 API 运行在 Cloudflare Workers。
- D1 是笔记、用户、关系、分享、设置和附件元数据的事实源。
- R2 只存附件、导出包和对象文件。
- `/api/v1/*` 提供 Memos 兼容 API 子集。
- 人的生产访问交给 Cloudflare Access；脚本、Memos-compatible 客户端和 MCP 使用 Access Service Token。

不要把 FlareMo 改成 VPS、Docker、Postgres、Node 常驻服务或应用内 Bearer token 登录方案。

## 常用命令

```bash
pnpm install
pnpm check
pnpm test
pnpm build
pnpm verify
pnpm backup:drill
```

本地开发：

```bash
pnpm migrate:local
pnpm dev
```

生产部署：

```bash
pnpm verify
pnpm migrate:remote
pnpm deploy
```

只验证 Cloudflare 配置和打包：

```bash
pnpm deploy:dry-run
```

## 修改规则

- 改数据库结构时，先改 `packages/db/src/schema.ts`，再运行 `pnpm db:generate`。
- 生成的 SQL migration 必须提交到 `migrations/`。
- 业务访问数据必须通过 Drizzle 和 domain services，不要在路由里堆散装 SQL。
- `/api/v1/*` 是兼容层；新增字段或行为时同步检查 `packages/memos` 和 OpenAPI。
- `/api/app/*` 可以服务前端体验，但必须复用同一套 domain services。
- 前端只展示已经接上后端能力的入口；不要放未实现功能的按钮、菜单或文案。
- 生产访问边界是 Cloudflare Access；不要新增应用内访问令牌登录页。
- `Temp/` 是参考仓库目录，不能提交。
- 不使用 GitHub Actions 作为项目 CI；提交前在本地跑 `pnpm verify`。

## 验收口径

改动完成前至少跑：

```bash
pnpm verify
```

涉及部署、Wrangler、D1、R2 或 Access 的改动，还要跑：

```bash
pnpm deploy:dry-run
```

涉及 UI 的改动，要启动本地服务检查桌面和移动端：

```bash
pnpm dev
```

## 文档入口

- `README.md`：项目入口和部署入口。
- `docs/tech-stack.md`：确定的技术栈。
- `docs/architecture-notes.md`：架构和兼容边界。
- `docs/deploy.md`：人类部署指南。
- `docs/agent-deploy.md`：Agent 部署 runbook。
- `docs/release.md`：发版规则。
- `docs/maintenance.md`：维护、备份和恢复。
- `docs/memos-compatibility.md`：Memos 兼容矩阵。
- `ROADMAP.md`：稳定方向和公开任务池。

---
> Source: [realchendahuang/FlareMo](https://github.com/realchendahuang/FlareMo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
