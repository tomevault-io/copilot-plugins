## openclaw-docs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是 **OpenClaw 中文文档站**，基于 VitePress 构建的静态文档网站，部署在 Azure Static Web Apps。文档内容覆盖 OpenClaw（原名 ClawdBot）的安装教程、源码剖析和 AI 框架解析。

## 常用命令

```bash
npm run docs:dev      # 本地开发（监听 0.0.0.0，局域网可访问）
npm run docs:build    # 构建静态产物到 docs/.vitepress/dist/
npm run docs:preview  # 本地预览构建结果
```

**内容转换脚本**（将外部 MDX 文档批量转为 VitePress Markdown）：

```bash
node scripts/convert-mdx.mjs <src-dir> <dest-dir>
# 示例：node scripts/convert-mdx.mjs D:/my_code/projects/openclaw/docs/start docs/tutorials/getting-started
```

## 架构要点

### 目录结构

```
docs/
├── index.md                          # 首页
├── tutorials/                        # Track 0：安装教程（按主题分子目录）
│   ├── getting-started/
│   ├── installation/
│   ├── gateway/
│   ├── channels/
│   ├── providers/
│   ├── concepts/
│   ├── tools/
│   ├── automation/
│   └── help/
├── beginner-openclaw-guide/          # Track A：完整工程主线（59 篇）
└── beginner-openclaw-framework-focus/ # Track B：AI 核心框架（22 篇）
docs/.vitepress/
├── config.mts                        # 站点核心配置（导航、侧边栏、SEO）
└── theme/
    ├── index.ts                      # 自定义主题入口
    └── style.css                     # 全局样式覆盖
```

### 侧边栏自动生成机制

`config.mts` 中的 `listMdItems(dir)` 函数会**自动扫描目录**并生成侧边栏：
- 文件按数字前缀排序（如 `01-xxx.md` < `02-xxx.md`）
- 标题从文件的 H1 自动提取，并去掉编号前缀
- 因此**新增文档只需放入对应目录**，侧边栏自动更新

`filterByRange(items, min, max)` 用于将 `beginner-openclaw-guide/` 和 `beginner-openclaw-framework-focus/` 的文章按编号分组到不同侧边栏阶段。

### {{ }} 转义问题

`vitepress-plugin-mermaid` 会覆盖 VitePress 默认的代码高亮函数，导致 Markdown 代码块中的 `{{ }}` 被 Vue 模板引擎解析报错。

`config.mts` 中通过**包装 `fence` 和 `code_inline` 渲染器**来重新应用转义，避免 Vue SSR 编译错误。新增文档中如有 `{{ }}` 语法，**不需要手动转义**，构建层面已自动处理。

### 部署

推送到 `main` 分支后，GitHub Actions 自动触发 Azure Static Web Apps 部署（配置见 `.github/workflows/`）。构建输出目录为 `docs/.vitepress/dist/`。

站点 URL：`https://openclaw-docs.dx3n.cn`

### SEO 配置

`config.mts` 中包含完整的 SEO 设置：
- 每页自动生成 canonical URL 和 OG meta 标签（通过 `transformHead`）
- 首页嵌入 JSON-LD 结构化数据（WebSite / Organization / FAQPage）
- Sitemap 自动生成

---
> Source: [yeuxuan/openclaw-docs](https://github.com/yeuxuan/openclaw-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
