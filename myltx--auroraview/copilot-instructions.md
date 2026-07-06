## auroraview

> - 仓库采用 npm workspaces，将应用划分为 `packages/*`。`packages/main` 存放 Electron 主进程代码，`packages/preload` 托管预加载脚本，`packages/renderer` 用于界面实现（可通过 `npm run create-renderer` 生成并位于同级 src 目录）。

# Repository Guidelines

## Project Structure & Module Organization
- 仓库采用 npm workspaces，将应用划分为 `packages/*`。`packages/main` 存放 Electron 主进程代码，`packages/preload` 托管预加载脚本，`packages/renderer` 用于界面实现（可通过 `npm run create-renderer` 生成并位于同级 src 目录）。
- `packages/integrate-renderer` 与 `packages/electron-versions` 提供 CLI 与版本辅助逻辑，构建脚本位于根级 `packages/dev-mode.js`、`packages/entry-point.mjs`。静态资源与签名文件放入 `buildResources`，端到端测试位于 `tests/e2e.spec.ts`。

## Build, Test, and Development Commands
- `npm start`：通过 `packages/dev-mode.js` 启动开发模式并监听热更新。
- `npm run build`：在所有 workspace 内执行 `vite build` 或对应 `build` 脚本，生成 `dist/`。
- `npm run compile`：先运行 `build`，随后触发 `electron-builder` 生成安装包；可附加 `--dir -c.asar=false` 方便调试。
- `npm run test`：使用 Playwright 针对 `npm run compile` 产物执行端到端用例（默认 `tests/e2e.spec.ts`）。
- `npm run typecheck`：在各 package 内运行 `tsc --noEmit`，确保严格模式未报错。
- `npm run init`：顺序执行 `create-renderer`、`integrate-renderer` 并重新安装依赖，用于快速新建界面包。

## Coding Style & Naming Conventions
- `.editorconfig` 规定 UTF-8、LF、尾随换行、无行尾空格（Markdown 允许保留），统一使用两个空格缩进。
- TypeScript 采用 `module/target: ESNext` 与严格模式，各包通过路径别名 `@app/*` 互相引用；自定义类型请放入 `types/*.d.ts` 并在 `tsconfig` 的 `paths` 中声明。
- 仓库默认不强制 lint/format，请沿用现有文件风格（例如保持函数导出命名使用小驼峰，类使用帕斯卡命名，常量使用 `ALL_CAPS` 或清晰前缀）。

## Testing Guidelines
- 端到端测试基于 Playwright（`@playwright/test`），推荐在本地先执行 `npm run compile && npm run test`，以便复现 CI 环境。
- 当为 package 编写单测时，可使用各自技术栈的测试框架，并将文件命名为 `*.spec.ts`/`*.test.ts`，这些文件会被 `tsconfig` 排除在构建之外。
- 提交前请确保核心流程（启动、更新、自动下载）在至少一个桌面平台通过冒烟测试，并在 PR 描述中注明验证平台。

## Commit & Pull Request Guidelines
- Git 历史保持动词开头、简洁描述（示例：`Add image selector and fullscreen functionality`）。遵循此风格，必要时在正文补充动机与影响面。
- PR 必须包含变更背景、测试结果与风险提示；涉及 UI 或安装器变更时附上屏幕截图或打包日志，并关联相关 issue。
- 变更前若计划引入额外依赖或影响打包流程，请先开启讨论或草稿 PR，以便维护者审阅。

## Environment & Security Notes
- 运行模式通过 Vite 的 `import.meta.env` 注入，仅 `VITE_` 前缀变量会暴露给渲染进程；新增变量需同步到 `types/env.d.ts` 的 `ImportMetaEnv`。
- 本模板遵循 Electron 官方安全指引：保持 `contextIsolation` 为 `true`，所有文件访问与 IPC 都应封装在 `packages/preload/src` 暴露的 API，避免在渲染层使用 Node 原语。

---
> Source: [myltx/AuroraView](https://github.com/myltx/AuroraView) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
