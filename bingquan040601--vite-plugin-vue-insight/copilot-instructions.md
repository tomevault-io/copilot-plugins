## vite-plugin-vue-insight

> 此文件为 Claude Code（claude.ai/code）在本仓库中工作时提供指导。

# CLAUDE.md

此文件为 Claude Code（claude.ai/code）在本仓库中工作时提供指导。

## 项目概述

`vite-plugin-vue-insight` 是一个 Vite + Vue 3 调试插件。按住 `Alt+Shift`（或 Mac 上的 `⌥+Shift`）点击页面元素即可高亮 DOM、在控制台打印组件状态，并自动在编辑器中打开对应源码文件。

## 常用命令

根目录（库本身）：

```bash
# 无构建步骤 — 以源码形式直接发布
```

Demo 应用（`demo/`）：

```bash
cd demo
npm run dev      # 启动 Vite 开发服务器
npm run build    # 生产构建
npm run preview  # 预览生产构建
```

发布到 npm（需登录）：

```bash
npm publish
```

## 架构

插件核心代码集中在 `src/vite-plugin.js`（约 447 行），通过 `src/index.js` 导出 `vueInsightPlugin()`。

### 两阶段设计

**阶段一 — 构建时注入**（`transform` 钩子）：
- 在 Vite transform 阶段拦截 `.vue` 文件
- 用 `@vue/compiler-sfc` 解析出 `<template>` 块
- 逐字符遍历模板内容，识别 HTML 标签起始位置
- 跳过组件标签（PascalCase）、虚拟标签（`<template>`、`<slot>`、`<component>`）、注释和闭合标签
- 在每个原生 HTML 标签上注入 `data-v-insight-file`、`data-v-insight-abspath`、`data-v-insight-line`、`data-v-insight-component` 属性
- 行号通过统计模板内容中的换行符数量，加上 SFC 中的基础偏移量计算得出

**阶段二 — 运行时注入**（`transformIndexHtml` 钩子）：
- 向 `index.html` 注入内联 `<script type="module">`（通过 `body-prepend`）
- 无需外部客户端打包 — 整个运行时以模板字符串形式内联（`inspectorClientCode`）

### 客户端运行时（`inspectorClientCode` 内联脚本）

注入的客户端脚本包裹在 IIFE 中，实现以下功能：

| 功能 | 实现方式 |
|---|---|
| **激活** | `keydown`/`keyup` 监听 `Alt+Shift`（Mac 下显示 `⌥+Shift`） |
| **悬停高亮** | 蓝色边框浮层（`position:fixed`、`pointer-events:none`），通过 `requestAnimationFrame` 节流的 `mousemove` 更新 |
| **阻止选中** | 检查模式下 `selectstart` 事件调用 `preventDefault()` |
| **点击高亮** | 红色脉冲边框浮层 + CSS `@keyframes __v-insight-pulse` 动画（持续 `highlightDuration` 毫秒，默认 4000ms） |
| **打开编辑器** | `fetch('/__open-in-editor?file=路径:行号:列号')` → 服务端中间件处理；回退方案：`location.href` 使用编辑器 URL scheme |
| **组件状态** | 读取 `el.__vueParentComponent` 提取 props 和响应式数据（ref/reactive），通过 `console.log` 打印 |
| **可分享链接** | 生成 `vscode://file/...`（或 `cursor://`、`webstorm://` 等）并自动复制到剪贴板 |

### 服务端中间件（`configureServer` 钩子）

处理 `/__open-in-editor` 请求，以分离子进程方式启动配置的编辑器：
- `vscode` → `code -g file:line:col`
- `cursor` → `cursor -g file:line:col`
- `vscode-insiders` → `code-insiders -g file:line:col`
- `webstorm` → `webstorm --line line --column col file`

### 边界情况：文件路径中的冒号

`file` 查询参数格式为 `filePath:line:col`。Windows 下绝对路径以 `C:` 开头，按 `:` split 后 `parts[0]` 只能拿到 `C`。这是已知限制 — 插件目前优先使用相对路径（来自 `file` 属性），绝对路径（来自 `abspath`）作为回退。

### window 配置对象

IIFE 运行前设置 `window.__VUE_INSIGHT__`，包含 `editor`、`attrPrefix`、`modifiers`、`highlightColor` 和 `highlightDuration`。客户端运行时从中读取配置。

## Demo 目录

`demo/` 是一个独立的 Vite + Vue 3 应用，通过 `file:..` 本地链接插件。作为开发和手动测试环境使用。运行 `cd demo && npm run dev` 即可测试插件改动。

---
> Source: [bingquan040601/vite-plugin-vue-insight](https://github.com/bingquan040601/vite-plugin-vue-insight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
