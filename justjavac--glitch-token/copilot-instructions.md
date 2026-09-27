## glitch-token

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.
<!-- END:nextjs-agent-rules -->

# 开发者文档

## 技术栈

- Next.js `16.3.0-preview.6`（App Router，全静态 SSG）+ React 19 + TypeScript
- Tailwind CSS v4 + shadcn/ui（@base-ui/react 变体）
- 暗色/明亮双主题（next-themes，默认跟随系统）；中英双语（`/zh`、`/en`，`src/proxy.ts` 按 Accept-Language 重定向）
- 数据：`src/data/`（内置静态数据，无后端）

## 命令

```bash
# 本机通过 corepack 使用 pnpm（如 pnpm 不在 PATH，可用 corepack pnpm 代替）
corepack pnpm install
corepack pnpm dev      # 开发
corepack pnpm build    # 构建（全部页面静态预渲染，含数据校验）
corepack pnpm start    # 生产模式
corepack pnpm lint     # ESLint
```

## 站点 URL

生产构建必须设置 `SITE_URL`（用于 canonical/hreflang/sitemap/OG 的绝对 URL）。正式域名为 `https://glitch-token.jjc.fun`，已在 Vercel 项目环境变量中配置；本地生产构建用：

```bash
SITE_URL=https://glitch-token.jjc.fun corepack pnpm build
```

开发环境未设置时使用 `http://localhost:3000`；生产环境缺失或格式无效会直接终止构建，避免搜索引擎收录 localhost URL。

## 结构

- `src/app/[lang]/` — 双语路由（首页目录 + `tokens/[id]` 详情页 + `models/[id]` 聚合页，全部 SSG）
- `src/app/tokens.json/`、`src/app/llms.txt/`、`src/app/llms-full.txt/` — 面向程序/LLM 的静态输出
- `src/app/sitemap.ts`、`src/app/robots.ts`、`src/app/icon.svg`、`public/og.png` — SEO 与站点图标/分享图
- `src/proxy.ts` — Next 16 的 proxy（原 middleware）：语言协商与重定向
- `src/data/` — `types.ts` / `models.ts` / `tokens.ts` / `taxonomy.ts` / `validate.ts` 数据层
- `src/i18n/` — UI 文案字典（zh/en）
- `src/components/` — 目录筛选（`catalog-filters.tsx`，Select）、token 卡片、主题/语言切换、GitHub 链接、复制按钮等

## 数据约定

- 所有面向用户的文案双语（`Localized = { zh, en }`）
- `behavior.link` 为来源链接，`behavior.repro` 为在线复现/演示链接；所有链接必须来自实际抓取的一手来源，逐字核实，不得凭记忆构造 URL
- 修改 `src/data/` 后 `build` 会执行 `validateData()`：交叉引用、URL 合法性、双语完整性、展示形式错误会终止构建

## 部署

push 到 `main` → Vercel（GitHub 集成）自动部署到 <https://glitch-token.jjc.fun>。

---
> Source: [justjavac/glitch-token](https://github.com/justjavac/glitch-token) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
