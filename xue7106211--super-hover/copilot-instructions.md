## super-hover

> 本文件为 AI 助手（Claude / Kiro 等）在本仓库中工作时提供指引。请用中文回复。

# CLAUDE.md

本文件为 AI 助手（Claude / Kiro 等）在本仓库中工作时提供指引。请用中文回复。

## 项目概述

这是一个 **Super Hover 的工程化 Demo 展示项目**。Super Hover 是一个超轻量库，每一帧用 `requestAnimationFrame` + `elementFromPoint` 做命中检测，从而在**滚动时、元素移动时**也能持续追踪指针下方的元素（原生 `:hover` 在滚动时会被浏览器跳过更新）。

本 Demo 用一个可滚动列表演示其核心价值：鼠标不动、滚动列表时，高亮项会跟随指针实时更新，右侧面板同步显示当前激活项与指针坐标。

## 技术栈

- **构建工具**：Vite 8
- **框架**：React 19 + TypeScript 5
- **样式**：Tailwind CSS v4（通过 `@tailwindcss/vite` 插件，无需 `tailwind.config.js`）
- **核心库**：`super-hover`（使用其 React 适配器 `super-hover/react`）
- **包管理器**：npm

## 常用命令

```bash
npm install      # 安装依赖
npm run dev      # 启动开发服务器（http://localhost:5173）
npm run build    # 类型检查 (tsc -b) + 生产构建
npm run preview  # 预览生产构建产物
```

> 注意：`npm run dev` / `preview` 是长时运行进程，不要在自动化流程里阻塞式执行。

## 目录结构

```
src/
  main.tsx                  # React 入口，挂载 <App />
  index.css                 # 全局样式，Tailwind v4 入口（@import "tailwindcss"）
  App.tsx                   # 主组件：接入 Super Hover，管理激活项与指针状态
  data.ts                   # 示例数据（80 条 Album），含 Album 类型
  components/
    StatusPanel.tsx         # 纯展示组件，反映当前激活项与指针坐标
vite.config.ts              # Vite 配置（react + tailwindcss 插件）
tsconfig.*.json             # TS 工程引用配置（app / node 分离）
```

## 核心实现要点

理解和修改本项目时，请牢记以下关键机制：

1. **`useSuperHoverRef` 返回的是 callback ref**：把它直接挂在可滚动容器（`overflow-auto` 的元素）上即可，库会在节点挂载时自动接线，不需要手动管理 `ref.current`。这个挂载的容器就是 Super Hover 的 `root`。

2. **可激活元素的标记**：列表项需要加 `data-super-hover` 属性才会被命中。激活时库会自动加上 `data-super-hover-active`，**不要手动控制这个属性**。

3. **样式靠属性变体驱动**：用 Tailwind v4 的 `data-[super-hover-active]:...` 变体做高亮，不要用 JS 切 class。

4. **事件回调拿到的是 DOM 元素**：`onEnter / onLeave / onMove` 的 `event.detail.current` 是 DOM 元素而非业务数据。需要业务数据时，从元素上读自定义属性反查。本项目用 `el.dataset.superHoverId` 反查 `Map`。
   - 易错点：`data-super-hover-id` 在 `dataset` 中是 **camelCase** 的 `superHoverId`。

5. **`onMove` 默认关闭**：只有传了 `onMove` 才会监听 move 事件（出于性能考虑）。坐标在 `event.detail.x / event.detail.y`（视口坐标）。

## 编码约定

- TypeScript 开启 `strict`，类型导入用 `import type`（`verbatimModuleSyntax` 已启用）。
- 组件用函数式 + 命名导出；纯展示组件不持有状态，通过 props 接收数据。
- 修改后请运行 `npm run build` 验证类型检查与构建均通过。
- 样式优先使用 Tailwind 工具类；自定义全局样式放在 `src/index.css`。

## 无障碍提示

Super Hover 会在滚动/布局变化时频繁改变界面，对动画敏感的用户可能不适。若用于生产，请尊重 `prefers-reduced-motion`：可结合 `useSuperHoverRef` 的 `enabled` 选项在需要时禁用。

## 参考

- 官方文档：https://super-hover.danielpetho.com/
- 源码：https://github.com/danielpetho/super-hover

---
> Source: [xue7106211/super-hover](https://github.com/xue7106211/super-hover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
