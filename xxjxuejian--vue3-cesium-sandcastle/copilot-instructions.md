## vue3-cesium-sandcastle

> 这是一个基于 Vue 3、TypeScript、Vite 和 Cesium 的示例应用。主要源码位于 `src/`：`views/` 存放按类别组织的 Cesium 示例页面，`components/` 存放通用 UI 与 Cesium 组件，`hooks/` 存放可复用的 Composition API 逻辑，`router/` 和 `config/menu.ts` 负责导航，`stores/` 存放 Pinia 状态，`utils/` 存放通用工具。静态 mock 数据和 Cesium 资源位于 `public/mock/` 与 `public/SampleData/`。工具配置位于根目录，包括 `vite.config.ts`、`uno.config.ts`、`eslint.config.js` 和 TypeScript 配置。

# Repository Guidelines

## Project Structure & Module Organization

这是一个基于 Vue 3、TypeScript、Vite 和 Cesium 的示例应用。主要源码位于 `src/`：`views/` 存放按类别组织的 Cesium 示例页面，`components/` 存放通用 UI 与 Cesium 组件，`hooks/` 存放可复用的 Composition API 逻辑，`router/` 和 `config/menu.ts` 负责导航，`stores/` 存放 Pinia 状态，`utils/` 存放通用工具。静态 mock 数据和 Cesium 资源位于 `public/mock/` 与 `public/SampleData/`。工具配置位于根目录，包括 `vite.config.ts`、`uno.config.ts`、`eslint.config.js` 和 TypeScript 配置。

## Build, Test, and Development Commands

使用 PNPM 管理依赖；`pnpm-lock.yaml` 已提交到仓库。

- `pnpm install`：根据 lockfile 安装依赖。
- `pnpm dev`：启动 Vite development server。
- `pnpm build`：执行 TypeScript 检查，并在 `dist/` 生成 production build。
- `pnpm preview`：本地预览已构建的应用。
- `pnpm lint:eslint`：修复 Vue 和 TypeScript 文件中的 ESLint 问题。
- `pnpm lint:prettier`：格式化源码、配置和 Markdown 文件。
- `pnpm lint:stylelint`：修复 CSS、SCSS、Vue 和 HTML 样式问题。

## Coding Style & Naming Conventions

遵循现有 Vue Composition API 和 TypeScript 风格。Prettier 使用 2 空格缩进、分号、双引号、ES5 trailing commas，以及 100 字符行宽。Vue 组件文件和目录优先使用 PascalCase，函数和变量使用 camelCase，可复用 hooks 使用 `useXxx` 命名。Cesium 初始化和封装应放在 `components/Cesium/`、`core/cesium/` 或相关 hooks 中。

## Testing Guidelines

当前未配置专用 test runner。提交前请运行 `pnpm build` 和相关 lint 命令。涉及行为变更时，需要在 `pnpm dev` 中手动验证受影响的 Cesium 页面；如涉及资源加载，也要确认 `public/SampleData/` 可正常访问。后续如添加测试，请使用 `*.test.ts` 或 `*.spec.ts` 命名。

## Commit & Pull Request Guidelines

由于当前环境将该仓库标记为 dubious ownership，无法检查 Git history。提交信息请使用清晰的祈使句，例如 `Add camera flight demo` 或 `Fix Cesium token handling`。Pull Request 应包含简短说明、已执行的验证步骤、相关 issue 链接；涉及 UI 或 Cesium 可视变化时，请附截图或录屏。

## Security & Configuration Tips

不要提交个人 Cesium Ion token 或本地 secret。环境相关值放在 `.env.local` 中；Cesium 访问使用 `VITE_CESIUM_ION_TOKEN`。GitHub Pages 部署也需要通过 repository secrets 提供同名 token。

---
> Source: [xxjxuejian/Vue3-Cesium-SandCastle](https://github.com/xxjxuejian/Vue3-Cesium-SandCastle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
