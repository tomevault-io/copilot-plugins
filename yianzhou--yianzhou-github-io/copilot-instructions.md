## yianzhou-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

```bash
yarn start        # 启动本地开发服务器，地址为 http://localhost:3000
yarn build        # 构建生产环境静态站点，输出到 /build/ 目录
yarn serve        # 本地预览构建产物
yarn clear        # 清除 Docusaurus 缓存（构建异常时使用）
yarn deploy       # 部署到 GitHub Pages（通常由 CI 自动执行）
```

## 架构说明

本项目是基于 **Docusaurus v3** 的静态文档站点，部署在 GitHub Pages，域名为 `yianzhou.com`。

### 多实例文档结构

站点使用了多个 Docusaurus `docs` 插件实例，每个实例对应 `/docs/` 下的一个顶级目录。每个实例在 `docusaurus.config.js` 中单独配置，并在 `sidebars.js` 中有对应的侧边栏定义。**新增文档板块时，需要同时修改 `docusaurus.config.js`（添加插件实例）和 `sidebars.js`（添加侧边栏配置）。**

### 内容

所有内容以 Markdown/MDX 格式存放在 `/docs/` 目录下，侧边栏根据文件系统结构自动生成。原生支持 Mermaid 图表，并在 `docusaurus.config.js` 中为 Dart 和 Swift 配置了自定义语法高亮。

### 样式

自定义样式覆盖写在 `/src/css/custom.css` 中，通过 Infima CSS 变量控制亮色/暗色主题的色彩。

### 部署

向 `main` 分支推送代码会触发 `.github/workflows/deploy.yml`，自动构建并将产物推送到 `gh-pages` 分支。自定义域名通过 `/static/CNAME` 配置。

### 回答偏好

请称呼用户为安哥。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yianzhou)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yianzhou)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
