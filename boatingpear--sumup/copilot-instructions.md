## sumup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

个人知识库/文档站点，使用 VitePress 构建，部署在 GitHub Pages。内容涵盖前端、后端（Java）、AI 资源、个人日记和生活类笔记。

## 常用命令

```bash
# 安装依赖（使用 pnpm）
pnpm install

# 启动开发服务器（端口 8081，监听所有网络接口）
pnpm dev
# 或
pnpm docs:dev

# 构建生产版本
pnpm build
# 或
pnpm docs:build

# 预览生产构建
pnpm preview
# 或
pnpm docs:preview
```

没有配置 lint 和 test 命令。

## 整体架构

### 目录结构

```
docs/
├── .vitepress/
│   ├── config.ts          # 主配置：中文站点，base: /SumUP/，本地搜索，markdown 容器自定义标签
│   ├── nav/index.ts       # 顶部导航：首页、例子、前端、后端、资源、其他
│   ├── sidebar/           # 侧边栏按主题模块化组织
│   │   ├── index.ts       # 侧边栏入口，汇总所有子模块
│   │   ├── frontend/      # 前端相关侧边栏（React基础、浏览器模型、工具、微信小程序）
│   │   ├── server/        # 后端相关（Java基础：初级/中级/高级/进阶）
│   │   ├── resource/      # AI资源侧边栏（Agent、模型、RAG、Dify、ComfyUI 等）
│   │   ├── other/         # 其他侧边栏（健康、Trae）
│   │   ├── listOfEvents/  # 个人流水账（按年/月组织的日记）
│   └── theme/
│       ├── index.ts       # 自定义主题入口，继承 DefaultTheme
│       ├── components/
│       │   ├── Layout.vue  # 自定义布局：View Transitions API 实现暗色模式切换动画
│       │   └── backtotop.vue
│       ├── custom.css     # 首页 features 布局、blockquote 样式、卡片悬停效果
│       └── style/         # CSS 模块：blockquote, blur, custom-block, marker, sidebarIcon, var, vp-code-title
└── src/                   # 所有 Markdown 文档内容
    ├── index.md           # 首页（hero 布局）
    ├── examples/          # 例子/创意想法
    ├── frontend/          # 前端知识（React, 浏览器API, 工具集成, 微信小程序）
    ├── server/            # 后端知识（Java基础分层教学）
    ├── resource/          # AI相关资源与学习路线
    ├── other/             # 生活类（健康、理财、护肤、菜谱、情感）
    ├── listOfEvents/      # 个人流水账日记
    └── public/            # 静态资源（favicon, 图片, SVG）
```

### 关键设计

- **站点 base 路径**为 `/SumUP/`，匹配 GitHub Pages 仓库名。所有链接和资源路径需考虑此前缀。
- **侧边栏组织**：每个独立主题有独立的 `sidebar/<topic>/index.ts` 文件，在 `sidebar/index.ts` 中按路由路径统一注册。
- **导航栏**中一些分类（如"流水账"）已被注释隐藏，但侧边栏路由仍保留。
- **`ignoreDeadLinks: true`** 在配置中被启用，死链接不会导致构建失败。
- **自定义主题**在默认主题基础上增加了：暗色模式切换的 View Transitions 动画、返回顶部组件、多种 CSS 定制样式。
- **搜索**使用 VitePress 本地搜索（`provider: 'local'`），带中文界面翻译。

### CI/CD

GitHub Actions 工作流（`.github/workflows/deploy.yml`）在 push 到 `main` 分支时自动构建并部署到 GitHub Pages。使用 pnpm v10.28.0，Node 20。

## 内容编写规范

新增文档时的操作模式：
1. 在 `docs/src/<category>/` 下创建 `.md` 文件
2. 在 `docs/.vitepress/nav/index.ts` 中添加导航条目（如需要出现在顶部导航）
3. 在 `docs/.vitepress/sidebar/<category>/index.ts` 中添加侧边栏条目
4. 在 `docs/.vitepress/sidebar/index.ts` 中确保侧边栏路由已注册

---
> Source: [boatingPear/SumUP](https://github.com/boatingPear/SumUP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
