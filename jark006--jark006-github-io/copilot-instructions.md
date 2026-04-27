## jark006-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

这是一个基于 **Jekyll** 的静态博客站点，部署在 **GitHub Pages** 上。站点语言为简体中文，采用 GitHub Dark 风格设计，支持明暗主题切换。

- **站点地址**: https://jark006.github.io
- **所有者**: JARK006

## 技术栈

| 组件 | 技术 |
|------|------|
| 静态站点生成器 | Jekyll (Ruby) |
| Markdown 处理器 | kramdown (GFM 输入) |
| 语法高亮 | Rouge (GitHub Dark 主题) |
| CSS 预处理器 | Sass/SCSS |
| 前端交互 | Alpine.js (CDN) + 原生 JS |

## 常用命令

仓库中未提交 `Gemfile` 和 `Gemfile.lock`（在 `.gitignore` 中），本地开发需先初始化：

```bash
# 首次本地开发
bundle install

# 本地预览（自动构建 + 热重载）
bundle exec jekyll serve --livereload

# 仅构建
bundle exec jekyll build
```

## 目录结构

```
├── _config.yml           # Jekyll 配置（分页、TOC、kramdown 等）
├── index.html            # 首页（使用 home 布局，带分页）
├── archive.html          # 按年份归档的文章列表
├── tags.html             # 标签云 + 按标签分组的文章
├── about.md              # 关于页面
├── 404.html              # 404 页面
├── search.json           # 客户端搜索的 JSON 索引
├── _layouts/             # 页面模板
│   ├── default.html       # 主 HTML 模板（含 Alpine.js 主题/进度条/灯箱组件）
│   ├── home.html          # 首页文章列表（卡片布局 + 分页）
│   ├── page.html          # 简单内容页
│   ├── post.html          # 文章页（含 TOC + 上下篇导航）
│   └── post_notitle.html  # 无标题文章页
├── _includes/            # 可复用组件
│   ├── header.html        # 顶部导航（头像、搜索、主题切换）
│   ├── footer.html        # 页脚（版权、GitHub、RSS 链接）
│   └── toc.html           # 目录组件（桌面侧边栏 + 移动端下拉）
├── _sass/                # SCSS 样式
│   ├── _variables.scss    # 设计令牌（颜色、间距、字体）
│   ├── _base.scss         # 重置、排版、全局样式
│   ├── _layout.scss       # 头部、底部、容器、搜索栏
│   ├── _post.scss         # 文章卡片、分页、归档、标签
│   ├── _code.scss         # Rouge 语法高亮
│   ├── _components.scss   # 滚动动画、TOC、灯箱、代码复制
│   ├── _dark-theme.scss   # 暗色主题变量
│   ├── _light-theme.scss  # 亮色主题变量
│   └── _animations.scss   # 关键帧动画
├── _posts/               # 博客文章（Markdown）
├── assets/
│   ├── css/style.scss     # SCSS 入口（导入所有 partials）
│   └── js/main.js         # JS（滚动动画、TOC 生成、代码复制、灯箱）
├── images/               # 静态图片资源
└── i/                    # 独立子应用（绕过 Jekyll 处理）
    ├── freezeit/          # Vue.js 应用
    └── lifetimeline/
```

## 关键配置 (`_config.yml`)

- **固定链接格式**: `/:title/`（仅标题，不含日期）
- **分页**: 每页 5 篇文章，路径 `/page:num/`
- **TOC**: 全站启用 (`toc: true`)
- **kramdown**: GFM 输入，TOC 级别 2-3，自动生成 ID，Rouge 高亮带行号
- **插件**: `jekyll-feed`, `jekyll-seo-tag`, `jekyll-paginate`

## 添加新文章

在 `_posts/` 目录下创建 Markdown 文件，命名格式为 `YYYY-MM-DD-slug.md`，文件头需包含：

```yaml
---
layout: post
title: "文章标题"
date: YYYY-MM-DD HH:MM:SS +0000
categories: [分类]
tags: [标签]
---
```

## CI/CD

`.github/workflows/pages.yml` 在推送到 `main` 分支时自动构建和部署：
1. `actions/jekyll-build-pages@v1` 构建站点
2. `actions/deploy-pages@v4` 部署到 GitHub Pages

## 样式开发

- 所有样式通过 `assets/css/style.scss` 导入 `_sass/` 中的 partials
- 输出格式为压缩模式（`:compressed`）
- 主题通过 CSS 自定义属性（变量）实现明暗切换
- 新增样式优先在对应的 `_sass/` partial 中添加

## 前端交互

- **Alpine.js**: 主题切换、阅读进度条、返回顶部、灯箱、搜索组件
- **原生 JS** (`assets/js/main.js`): 滚动触发动画、TOC 生成 + 滚动监听、代码块复制按钮、图片灯箱、Toast 通知

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jark006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
