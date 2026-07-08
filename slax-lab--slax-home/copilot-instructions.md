## slax-home

> Slax 官网，单 Astro 仓（2026-04 由原 monorepo 合并而来），路径划分如下：

# Slax 官网项目 - AI 操作指南

## 项目简介

Slax 官网，单 Astro 仓（2026-04 由原 monorepo 合并而来），路径划分如下：

- `/` — 主站（首页 + About + Privacy + Terms）
- `/blog/` — 博客
- `/reader/`、`/reader/changelog/`、`/reader/alternatives/` — Slax Reader 着陆页 + 发布日志 + 对比文章
- `/note/`、`/note/changelog/`、`/note/alternatives/` — Slax Note 同上

技术栈：Astro 6 + MDX + sitemap，无 Starlight、无客户端框架，默认零 JS。包管理器 pnpm。代码规范用 Biome，提交走 husky + lint-staged。部署目标 Cloudflare Pages。

## 常用操作速查

### 日常开发

| 用户说的话 | 执行命令 |
|---|---|
| 帮我运行/启动这个项目 | `pnpm dev` |
| 帮我安装依赖 | `pnpm install` |
| 帮我检查代码规范 | `pnpm lint` |
| 帮我修复代码规范问题 | `pnpm lint:fix` |
| 帮我格式化代码 | `pnpm format` |

### 打包与部署

| 用户说的话 | 执行命令 |
|---|---|
| 帮我打包 / 构建项目 | `pnpm build` |
| 帮我打包并部署到测试环境 | `pnpm deploy:dev` |
| 帮我在本地预览打包结果 | `pnpm build && pnpm preview` |

### Cloudflare 相关

| 用户说的话 | 执行命令 |
|---|---|
| 帮我登录 Cloudflare | `npx wrangler login` |
| 我想查看 Cloudflare 部署状态 | `npx wrangler pages deployment list --project-name=slax-home` |

## 项目结构说明

```
slax-home/
├── astro.config.mjs
├── public/                    # 静态资源（图片、og-default.png、robots.txt）
├── src/
│   ├── components/            # Nav、Footer、Logo、ProductCard 等共享组件
│   ├── content/               # Astro Content Collections
│   │   ├── blog/              # 博客文章（.mdx）
│   │   ├── reader-changelog/  # Reader 发布日志（.md，按日期 + 版本命名）
│   │   ├── note-changelog/    # Note 发布日志（同上）
│   │   ├── reader-alternatives/ # vs 对手对比文（.mdx）
│   │   └── note-alternatives/   # 同上
│   ├── content.config.ts      # collections 定义 + Zod schema
│   ├── layouts/Base.astro     # 全站 layout（SEO meta、JSON-LD、字体）
│   ├── lib/og.ts              # OG 图片渲染器（详见下方"OG 图片规则"）
│   ├── pages/                 # 路由
│   │   ├── og/                # OG endpoints（每页一个 .png.ts，构建期生成 PNG）
│   │   ├── blog/[slug].astro
│   │   ├── reader/index.astro · changelog.astro · alternatives/[slug].astro
│   │   └── note/...           # 同 reader 结构
│   └── styles/                # tokens.css + global.css
└── package.json
```

## 开发端口

`pnpm dev` 启动本地开发，单端口：http://localhost:4321

## 内容编辑指引

- **博客**：`src/content/blog/<slug>.mdx`，frontmatter 见 `content.config.ts` 的 `blog` schema（必填 `title` / `description` / `pubDate`）
- **Reader / Note 发布日志**：`src/content/<reader|note>-changelog/YYYY-MM-DD-vX.Y.Z[.platform].md`，frontmatter 必填 `version` / `date` / `platforms`（数组）
- **对比文章**：`src/content/<reader|note>-alternatives/<competitor>.mdx`
- **图片**等静态资源：`public/images/`（CDN 用 `https://static-cdn.slax.com/...`）
- **内链写法**：所有 locale 的内容文件里，站内链接一律写**不带 locale 前缀**的路径（如 `/reader/alternatives/pocket/`）。构建时 `scripts/remark-locale-links.mjs` 会按内容文件所在 locale 自动加前缀（目标页在该 locale 存在才加）。不要手写 `/ja/...`、`/es/...` 前缀。验证：`pnpm check:locale-links`（需先 build）

## 注意事项

- 部署前必须先登录 Cloudflare（`npx wrangler login`），只需登录一次
- `dist/` 目录是自动生成的，不要手动修改其中的文件
- 修改代码后如果需要部署到测试环境，直接执行 `pnpm deploy:dev` 即可，它会自动先打包再上传
- 提交前 husky + lint-staged 会跑 `biome check --write`，不要用 `--no-verify` 跳过

## 品牌与大小写规则

品牌名和专有名词在页面上必须保留标准大小写，不要用 `text-transform: uppercase` 把它们"美化"成全大写：

- Slax、Slax Reader、Slax Note —— 保持混合大小写
- iOS、macOS、iPadOS、tvOS —— 保留前缀小写
- GitHub、TikTok、LinkedIn 等同理

如果确实需要一个小号、宽字距的"眉标"样式，请使用 mono 字体 + 小 letter-spacing（如 `0.04em`），不要加 `text-transform: uppercase`。只有纯英文的泛用词（如 `RELEASE NOTES`、`FEATURED`）才适合全大写，且此时不能夹带品牌词。

## 排版偏好

- 博客正文**不要首字母放大（drop cap）**。首段保持略大一些的 lede 样式即可。

## OG 图片规则（社交分享卡片）

**每个新页面都必须有专属 OG 图。** 不要让页面回退到默认的 `/og-default.png`。社交平台（X、LinkedIn、Slack、Discord、iMessage、微信）依赖 1200×630 PNG/JPG，渲染卡片时如果链接没有专属图，体验会和站内调性脱节。

### 怎么做

1. **共享渲染器**：`src/lib/og.ts` 导出 `renderOg({ eyebrow, title, accentWord? })`，输出 1200×630 PNG Buffer，serif 标题 + 左上 Slax 标识 + 左下 mono 眉标，可指定一个词标绿斜体重音。
2. **每个新页面加一个 endpoint**：在 `src/pages/og/` 下放一个 `.png.ts` 文件，调用 `renderOg` 并返回 PNG Response。
   - 静态页（如 `/about/`、`/reader/changelog/`）→ `src/pages/og/<name>.png.ts`
   - 动态页（如 `/blog/[slug]/`）→ `src/pages/og/blog/[slug].png.ts`，用 `getStaticPaths()` 给每条内容生成。
3. **在页面 `<Base>` 里挂上**：`ogImage="/og/<name>.png"`。`Base.astro` 已经自动加好 `og:image:type/width/height/alt` 和 `twitter:image`。

### 不要做

- ❌ 不要为新页面只用默认图就发版
- ❌ 不要直接把营销 cover 图（hero、product shot）当 OG（构图比、字号、留白都不对）
- ❌ 不要在 `og.ts` 里用 SVG OG 输出（X / LinkedIn / Slack 不可靠地解 SVG）

### 验证清单

每次发布前抽 1-2 个新链接到这三个工具跑一遍：

- X：https://cards-dev.x.com/validator
- Facebook / Meta：https://developers.facebook.com/tools/debug/
- LinkedIn：https://www.linkedin.com/post-inspector/

或者直接在 Slack / Discord 私信粘链接看预览。

---
> Source: [slax-lab/slax-home](https://github.com/slax-lab/slax-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
