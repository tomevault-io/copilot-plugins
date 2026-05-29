## kuiper

> 本文档持续记录对本项目的开发习惯和约定，供 AI Agent 在执行任务时参考。

# AGENTS

本文档持续记录对本项目的开发习惯和约定，供 AI Agent 在执行任务时参考。

## 代码风格

- 使用**单引号**（single quote）
- 所有语句末尾**不加分号**（no semi）
- 使用**尾随逗号**（trailing commas: all）
- 使用 **2 空格缩进**（2-space indent）
- 仅在需要时手动执行 lint 和 format，不在保存时自动触发

## 技术栈

- pnpm 作为包管理器
- Electron 42.x + electron-vite 5.x
- Vite 8.x（通过 pnpm.peerDependencyRules 放行 peerDep 冲突）
- TypeScript 6.x
- Vue 3.x + Vue Router 5.x + Naive UI 2.x
- Pinia 3.x 用于全局状态管理，可复用逻辑优先使用 Pinia store 或 Vue composable
- oxlint 用于代码检查，oxfmt 用于代码格式化
- tsx 用于执行 scripts 目录下的 TypeScript 脚本
- electron-builder 用于 Windows 打包

## 前端组件约定

- 所有组件使用 **PascalCase**（大驼峰）命名，包括自定义组件和 NaiveUI 组件
- NaiveUI 组件在使用处显式 import，确保 VSCode 类型提示
- View 组件（`views/`）仅做布局编排，不包含复杂业务逻辑
- 可复用的 UI 片段拆分为独立子组件（`components/`）
- 跨组件共享的状态使用 Pinia store（`stores/`）
- 可复用逻辑封装为 Vue composable（必要时）
- 全局 UI 状态（如面板显隐、侧边栏折叠）统一放在 Pinia store 中，不在组件内用 `ref` 管理

## 项目约定

- 程序名为英文 `kuiper-box`，窗口标题为中文「柯伊伯方盒」
- GitHub 仓库：https://github.com/maanfa/kuiper
- 使用国内 npm 镜像加速依赖下载
- Electron 二进制缓存到 `.cache/electron` 目录
- 开发模式窗口标题追加 ` - [DevMode]` 标识
- `build` 命令用于页面构建（electron-vite build）
- `dist` 命令用于安装包构建（electron-builder），输出到 `release/win_unpacked`
- 清理命令：`pnpm clean`（构建产物）、`pnpm clean:out`（仅 out）、`pnpm clean:release`（仅 release）、`pnpm clean:cache`（electron 缓存）、`pnpm clean:all`（全部）
- Electron 打包输出命名格式：`KuiperBox-setup-win-x64-${version}.exe`
- 构建脚本统一放在 `scripts/` 目录，使用 `tsx` 执行
- 配置文件 `app.config.yml` 位于项目根目录（开发）或 exe 同目录（打包）
- Electron 二进制可能因 pnpm 构建脚本策略未自动下载，需在 `pnpm.onlyBuiltDependencies` 中明确列出 `electron`，或手动执行 `node node_modules/electron/install.js`
- 所有文件使用 LF 换行符
- VSCode 设置显式指定 TypeScript SDK 路径为 `node_modules/typescript/lib`

## 主进程特性

- 启动时关闭沙盒模式（`--no-sandbox`）
- 最小分辨率 1280x720
- 无菜单栏
- Ctrl+Shift+I / F12 打开 DevTools
- Ctrl+R 强制刷新页面
- 窗口关闭时自动保存位置和大小到 `app.config.yml`
- 启动时自动恢复上次窗口位置和大小

## 日志系统

- 使用项目内置的轻量封装 Logger（`src/main/logger.ts`），不引入社区日志包
- 同时输出到控制台 stdout 和日志文件
- 通过 `app.config.yml` 的 `logging` 配置控制日志级别和文件输出路径
- 日志文件按天滚动（`app-YYYY-MM-DD.log`）
- 编码统一使用 UTF-8，确保中文不乱码
- 打包模式下仅当 yml 中配置了 `logging.filePath` 时才输出日志文件

## 沟通语言

- 所有文档、注释、交互均使用中文

---
> Source: [maanfa/kuiper](https://github.com/maanfa/kuiper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
