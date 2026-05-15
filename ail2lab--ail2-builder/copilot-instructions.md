## ail2-builder

> > 本文件是写给 AI 编码助手（如 Claude Code）的项目说明。任何需要读代码或改代码的任务，请先阅读本文件。

# AIL2-Builder — Claude 项目指南

> 本文件是写给 AI 编码助手（如 Claude Code）的项目说明。任何需要读代码或改代码的任务，请先阅读本文件。

## 1. 项目定位

**AIL2-Builder** 是 AIL2（去中心化 AI 超层，Decentralized AI Layer 2）的 **官方网站 + 开发者平台 + 内容管理系统**。面向开发者、GPU 算力提供方、生态合作伙伴，承担以下职责：

- 品牌官网与营销登陆页（Ecosystem、Accelerator、Incubator、Research）
- 多链 GPU 计算平台展示（Ethereum / BSC / XLayer / Base / Mantle）
- 博客 / 文档 / FAQ（Prisma + PostgreSQL 驱动的 CMS）
- 订阅、联系表单等运营功能
- 8 语言国际化

## 2. 仓库结构

```
AIL2-Builder/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── [locale]/          # i18n 路由（8 语言）
│   │   ├── admin/             # 后台管理
│   │   └── api/               # Route Handlers（admin/posts/seed）
│   ├── components/            # React 组件库
│   ├── actions/               # Server Actions
│   ├── hooks/                 # 自定义 hooks
│   ├── lib/                   # 工具函数
│   ├── i18n/                  # next-intl 配置
│   ├── config/                # 站点配置
│   ├── assets/                # 本地资源
│   └── middleware.ts          # Next.js 中间件（locale 路由匹配）
├── prisma/
│   ├── schema.prisma          # 数据模型：Subscriber/Contact/Post/Category/Tag
│   └── seed.ts                # 数据库初始化脚本
├── messages/                  # i18n 翻译（8 语言 JSON）
├── public/                    # 静态资源（图片、PDF、SVG）
├── ecosystem.config.js        # PM2 部署配置（port 3004, 1GB 内存上限）
├── docker-compose.yml         # 本地开发/部署辅助
├── next.config.ts
├── tsconfig.json
├── eslint.config.mjs
├── postcss.config.mjs
└── package.json
```

**主要页面路由**：`accelerate`、`blog`、`contact`、`core`、`docs`、`ecosystem`、`faq`、`incubator`、`platform`、`research`。

## 3. 技术栈

| 维度 | 选型 |
|------|------|
| 框架 | **Next.js 16.0.7**（App Router）|
| UI | **React 19.2.0** |
| 语言 | **TypeScript 5**（strict mode）|
| 样式 | **TailwindCSS 4** + PostCSS |
| 数据库 | **PostgreSQL** via **Prisma 6.19.0** |
| i18n | **next-intl 4.4.0**（8 语言）|
| 富文本 | **TinyMCE React 6.3.0** |
| 包管理 | **pnpm 10.17.1** |
| 进程管理 | **PM2**（生产）|
| Node 目标 | ES2017 |
| 路径别名 | `@/*` → `src/*` |

**尚未配置**：测试框架（无 Jest/Vitest）、Prettier、Husky。

## 4. 开发工作流程

### 常用命令

```bash
pnpm install                    # 安装依赖（首选 pnpm）
pnpm dev                        # 开发服务器
pnpm build                      # 生产构建
pnpm start                      # 启动生产服务器
pnpm lint                       # ESLint 检查
pnpm generate:prisma            # 生成 Prisma 客户端（改 schema 后必须执行）
pnpm db:seed                    # 初始化数据库
pnpm redeploy                   # 完整重部署（install + build + PM2 restart）
```

### 数据库改动流程

1. 改 `prisma/schema.prisma`
2. `pnpm prisma migrate dev --name <change_name>`（本地）/ `prisma migrate deploy`（生产）
3. `pnpm generate:prisma`
4. 更新相关 Server Actions 与 API 路由

### i18n 流程

1. 在 `messages/<locale>.json` 中新增 key（至少先在 `en.json` 加好）
2. 所有 8 种语言都要补齐，否则前端可能 fallback 出错
3. 在组件中用 `useTranslations()` 读取

### 部署

- 进程由 **PM2** 管理，端口 `3004`，内存上限 1GB
- 热更新走 `pnpm redeploy`
- 目前**无 `.github/workflows/` 配置**（CI/CD 待补）

## 5. 给 AI 助手的关键约定

1. **中文文字**：文档、commit 正文、用户可见文案（除 i18n 消息文件外）默认中文；代码标识符、路由、API path 用英文。
2. **App Router 优先**：新页面放 `src/app/[locale]/<route>/page.tsx`，不要走 Pages Router。
3. **Server Components 默认**：只在需要浏览器 API / hooks / 状态时加 `"use client"`。
4. **Server Actions** 放 `src/actions/`，每个文件头部加 `"use server"`。
5. **Prisma 查询**：始终通过 `src/lib/` 里的 singleton 客户端（不要 `new PrismaClient()`），避免开发模式下连接泄漏。
6. **i18n 改动**：新增文案必须**同步更新所有语言文件**；漏翻译要么 fallback 英文要么明确 TODO。
7. **SEO**：项目最近密集优化 SEO（参见 HEAD commit），改动 `<head>`、metadata、sitemap、robots 前请读相关代码避免回归。
8. **TypeScript strict**：不要用 `any` 绕过类型；如必须，加 `// eslint-disable-next-line` 并注明原因。
9. **Lint 通过**：`pnpm lint` 必须清洁，不要 push 带警告的代码。
10. **测试**：目前无测试框架；涉及关键逻辑（数据库、支付、表单校验）时可提议引入 Vitest，但不要擅自全量铺开。
11. **Prisma 生成物**：`src/generated/` 在 `.gitignore` 中，不要提交。
12. **部署不触碰**：PM2 ecosystem 文件与生产重启命令由用户控制，非明确授权不要自动 `redeploy`。

## 6. 多仓库协作上下文

Claude 同时管理 `C:\PROJECT\AIL2\` 下 4 个 AIL2Lab 仓库：

| 目录 | 角色 |
|------|------|
| `AIL2DEVOPS/` | DevOps / 部署中枢 |
| `AIL2-Builder/` | **本仓库** — 官网前端 + CMS |
| `AIL2-ProveNode/` | 节点验证（Go，DBC 链）|
| `AIL2-ComputeNet/` | 分布式 AI 计算网络（Go + libp2p）|

**相关联动**：
- Builder 展示的算力、节点状态、网络指标，数据源可能来自 **ComputeNet / ProveNode** 的 HTTP 或 Prometheus 接口。修改 API 契约时请通知两侧。
- 部署脚本、PM2 配置统一约定应迁移到 `AIL2DEVOPS/deploy/builder/`。

## 7. 历史对话总结

**2026-04-19 首次对话**，为 AIL2Lab 建立多仓库工作区：

1. 用户要求创建 SSH 密钥对（邮箱 `official@ail2.org`），生成 `~/.ssh/id_ed25519_ail2`（ed25519，无密码短语）。
2. 公钥被添加到 GitHub（账户 `AIL2-coder`）。
3. 初次通过 GitHub 公开 API 列出了 AIL2-Builder 与 AIL2-ProveNode 两个 repo，完成克隆。
4. 随后用户补充了 AIL2-ComputeNet（私有或当时未公开列出）并完成克隆。
5. `~/.ssh/config` 新增 `Host github-ail2` 别名，所有 4 个 repo 的 origin 切换到 `git@github-ail2:AIL2Lab/<name>.git`，`git pull/push` 自动使用新密钥。
6. 额外拉取 `AIL2DEVOPS` 到工作区作为 DevOps 中枢。
7. 用户明确 Claude 今后同时管理 4 个 AIL2Lab 仓库。
8. **本次行动**：为 4 个 repo 各生成一份 CLAUDE.md，包含仓库结构、工作流程、AI 约定、多仓库上下文与对话历史。

## 8. 下一步建议

- 补 `.github/workflows/`：至少 lint + build 的 PR 检查
- 引入 Vitest + Testing Library 覆盖关键组件
- 加 Prettier + Husky pre-commit（统一格式）
- 数据库 migration 流程文档化
- 将 PM2 部署步骤迁移到 `AIL2DEVOPS/deploy/builder/`

---
> Source: [AIL2Lab/AIL2-Builder](https://github.com/AIL2Lab/AIL2-Builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
