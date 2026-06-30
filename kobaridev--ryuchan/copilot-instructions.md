## ryuchan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

RyuChan — 基于 Astro 5.0+ 的静态博客模板，部署到 Cloudflare Pages。支持在线发布文章和可视化配置管理，通过 GitHub App 认证直接提交变更到仓库。

## 常用命令

```sh
pnpm dev                  # 开发服务器 (astro dev)
pnpm build                # 完整构建 (类型检查 → astro build → pagefind 索引 → 复制到 public)
pnpm run search:index     # 首次运行或更新内容后生成搜索索引 (完整 build + 复制 pagefind)
pnpm run preview          # 本地预览构建产物 (build + wrangler dev)
pnpm run check            # Astro 类型检查
pnpm run deploy           # 构建并部署到 Cloudflare (wrangler deploy)
pnpm run cf-typegen       # 生成 Cloudflare Workers 类型
pnpm run prefetch:music   # 预取音乐时长数据
pnpm run search:clean     # 清理搜索索引
```

## 技术栈

- **框架**: Astro 5.9+ (SSG / `output: "static"`)
- **部署**: Cloudflare Pages (adapter: `@astrojs/cloudflare`)
- **样式**: Tailwind CSS 3 + daisyUI 4 + SCSS
- **交互**: React 19 (客户端组件，标记 `'use client'`)，zustand 状态管理
- **内容**: Markdown/MDX (remark + rehype 插件链)，Pagefind 全文搜索
- **认证**: GitHub App (JWT + Installation Token)，AES-256 加密 PEM 密钥存储在 sessionStorage
- **包管理**: pnpm
- **Lint**: ESLint (`@antfu/eslint-config`)

## 架构

### 路由结构

```
/                      首页，文章列表分页 ([...page].astro)
/blog/[slug]           文章详情
/blog/archives         归档页
/blog/tags / /categories / /search   标签/分类/搜索页
/write                 在线写作 (React SPA: WritePage.tsx)
/write?slug=xxx        编辑已有文章
/config                可视化配置管理 (React SPA: ConfigPage.tsx)
/about / /about-edit   关于页 / 在线编辑
/project / /projects-edit   项目页 / 在线编辑
/navigation            网站导航
/anime                 追番 (TMDB + Bilibili API)
/album                 相册
/photo-wall            照片墙
/friend                友链
/music                 音乐页 (Meting 播放器)
```

### 配置系统

整个网站的配置来源于 `ryuchan.config.yaml`，通过 `src/config.ts` 在构建时读取并导出为常量。自定义 Astro 集成 (`src/integration/updateConfig.ts`) 监听配置文件和翻译文件的变更以触发热更新。

`src/consts.ts` 处理 GitHub 仓库配置（从 `import.meta.env` 或 YAML 配置合并）。

### 在线编辑机制

`/write` 和 `/config` 页面通过 GitHub Content API 直接读写仓库文件：

1. 用户导入 GitHub App 的 `.pem` 私钥文件
2. `src/lib/github-client.ts` 使用 `jsrsasign` 签发 JWT，换取 Installation Token
3. Token 缓存在 sessionStorage，PEM 经 AES-256 加密后缓存
4. `src/lib/auth.ts` 封装认证流程，`src/components/write/hooks/use-auth.ts` (zustand) 管理客户端认证状态
5. 文件读写通过 GitHub Contents API (`putFile` / `readTextFileFromRepo`)，相册等批量操作用 Git Trees API

### 目录约定

| 目录 | 用途 |
|------|------|
| `src/content/blog/` | 博客文章 (.md/.mdx)，由 Astro Content Collections 管理 |
| `src/pages/` | Astro 页面路由 |
| `src/components/` | 组件。`write/` = React 管理页面，`admin/` = 相册/照片管理，`mdx/` = MDX 专用组件，`widgets/` = 通用 UI 组件 |
| `src/layouts/` | 页面布局 (BaseLayout.astro) |
| `src/lib/` | 工具库：GitHub API 客户端、认证、frontmatter 解析、相册服务等 |
| `src/data/` | 静态数据 (YAML/JSON/TS)，如导航、友链、项目、相册、音乐 |
| `src/interface/` | TypeScript 类型定义 |
| `src/styles/` | 全局 SCSS 样式 |
| `src/stores/` | zustand stores |
| `src/plugins/` | Astro/Remark 插件 |
| `src/i18n/` | 多语言翻译 (translations.yaml) |
| `public/` | 静态资源 |
| `scripts/` | 构建辅助脚本 (fetch-music-duration.mjs) |

### 路径别名 (tsconfig.json)

- `@/*` → `src/*`
- `@components/*` → `src/components/*`
- `@layouts/*` → `src/layouts/*`
- `@config` → `src/config.ts`
- `@interfaces/*` → `src/interface/*`
- `@utils/*` → `src/utils/*`
- `@data/*` → `src/data/*`

### 环境变量

前缀 `PUBLIC_` 或 `NEXT_PUBLIC_`（Astro 静态构建约定，客户端可用）：
- `PUBLIC_GITHUB_OWNER` / `PUBLIC_GITHUB_REPO` / `PUBLIC_GITHUB_BRANCH`
- `PUBLIC_GITHUB_APP_ID`
- `PUBLIC_GITHUB_ENCRYPT_KEY`（可选，用于加密缓存中的 PEM）

### 搜索

使用 Pagefind。`pnpm build` 和 `pnpm run search:index` 会自动生成索引并复制 `dist/pagefind/**` 到 `public/pagefind`。

### Cloudflare 部署

使用 `wrangler.jsonc` 配置，`@astrojs/cloudflare` adapter。兼容性标志 `global_fetch_strictly_public` 和 `nodejs_compat`。

---
> Source: [kobaridev/RyuChan](https://github.com/kobaridev/RyuChan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
