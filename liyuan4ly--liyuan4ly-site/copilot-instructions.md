## liyuan4ly-site

> > 给 Claude Code 的项目使用指南。每次启动会自动加载,保持简洁。

# CLAUDE.md — liyuan4ly-site

> 给 Claude Code 的项目使用指南。每次启动会自动加载,保持简洁。

## 项目本质

LIYUAN 的个人网站 `https://liyuan4ly.com`,Astro 静态站,
内容包含「关于我」「项目展示」「博客」三大模块,
搭配入场页 + 双主题(夕阳/月夜)。

部署在 Cloudflare Pages,push `main` 自动构建。

## 关键文件

- `src/pages/` — 文件路由,每个 `.astro` / `.md` 即一个页面
- `src/layouts/BaseLayout.astro` — 全站外壳(导航 / 主题切换 / 字体)
- `src/components/` — 可复用片段(Nav / ThemeToggle)
- `src/lib/theme.ts` — 主题切换运行时(applyTheme / setTheme / toggleTheme)
- `src/content/blog/*.md` — 博客文章 Content Collection
- `src/content.config.ts` — Content Collection schema(frontmatter 类型)
- `src/data/projects.json` — 项目展示数据(静态,未接 GitHub API)
- `src/styles/global.css` — Tailwind v4 + `@theme` token + `:root` / `[data-theme]` 主题变量
- `public/bg/entrance.webp` — 入场背景图(已是 WebP,勿替换为大尺寸 PNG)

## 协作方式

- 中文回复,代码注释中文
- 改代码前先解释意图,等我确认(学习 > 结果)
- 类比 Spring Boot / Vue3 讲新概念
- Astro 组件无响应式,构建产物纯静态——这是默认前提,不要按 Vue/React 思路加 state

## 样式系统约定

- 全站用 **Tailwind v4**(`@theme` block + `@apply`),不裸写 CSS 除非必要
- 主题色统一从 CSS 变量读(`--color-bg` / `--color-text` 等),
  **不要直接写 `bg-zinc-900` 这种硬编码**——破坏主题切换
- 字体:Inter(英文)+ Noto Sans SC(中文),已在 BaseLayout 加载

## 项目特化禁项

- 不主动改 `src/styles/global.css` 的 `@theme` token 和 `[data-theme]`
  变量定义——这是主题切换的契约,改一个值会影响全站
- 暂时不接 GitHub API 拉项目列表——当前 4 个项目静态 JSON 维护够用

## 决策索引

`docs/decisions/`:

- 0001 框架选 Astro 不选 Next.js / VitePress
- 0002 裸域作主、`www` 301 跳裸域
- 0003 项目数据用静态 JSON 不调 GitHub API
- 0004 双主题(夕阳/月夜)+ 入场页 + localStorage 首访引导
- 0005 引入 Tailwind v4 + 单图 + 主题色蒙层

改这些前先看对应 ADR,Context 里通常已评估过其它选项。

---
> Source: [liyuan4LY/liyuan4ly-site](https://github.com/liyuan4LY/liyuan4ly-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
