## xyfs-sys

> 你好！我是 Gemini CLI，你的 AI 软件工程师。

# Gemini CLI 说明文档

你好！我是 Gemini CLI，你的 AI 软件工程师。

这个文件是我理解当前项目并与你协作的蓝图。你可以在这里提供关于项目结构、技术栈、开发规范和目标的详细信息。信息越完善，我能提供的帮助就越精准、高效。

## 项目概述

`xyfs` 是一个名为 "小象心选·团购真的省" 的项目。从目录结构和依赖来看，这似乎是一个包含小程序、管理后台和网站的多端应用。

-   `packages/xyfs_admin`: 小程序管理端。
-   `packages/xyfs_client`: 小程序客户端。
-   `packages/xyfs_wyy`: 是一个英语学习网站或H5应用，使用了 Next.js。
-   `common/`: 存放各个包之间共享的代码、工具函数和样式。

## 技术栈

-   **包管理器**: [pnpm](https://pnpm.io/)
-   **小程序框架**: [Taro](https://taro.zone/)
-   **Web 框架**: [Next.js](https://nextjs.org/)
-   **代码规范**:
    -   Commitlint: 用于规范 Git 提交信息。
    -   ESLint: 用于代码风格检查。
    -   Husky: 用于设置 Git Hooks。
-   **发布工具**: [release-it](https://github.com/release-it/release-it)

## 项目脚本

以下是 `package.json` 中定义的一些关键脚本：

-   `pnpm typecheck`: 对所有 `@xyfs/*` 包执行 TypeScript 类型检查。
-   `pnpm iit`: 使用 `release-it` 发布新版本。
-   `pnpm update:project`: 使用 Taro CLI 更新所有小程序相关项目的配置。
-   `pnpm clean`: 清理所有项目的 `node_modules`、`dist` 和其他构建产物。

## 如何与我协作

你可以通过自然语言向我发出指令，例如：

-   **代码理解**: "解释一下 `packages/xyfs_client/src/pages/index/index.tsx` 文件的作用。"
-   **代码编写**: "在 `xyfs_admin` 中添加一个新页面，用于展示用户信息。"
-   **代码重构**: "重构 `common/utils/src/request.ts`，使用 `async/await` 替代 Promise。"
-   **依赖管理**: "为 `xyfs_wyy` 添加 `lodash` 依赖。"
-   **自动化任务**: "帮我写一个脚本，自动将 `___cfiles` 目录下的所有 `.csv` 文件合并成一个。"
-   **提交代码**: "我修改了 `xyfs_client` 的首页，帮我提交代码，提交信息是 'feat: 更新首页UI'。"

当你需要我执行任务时，请尽量提供清晰、明确的指令。期待与你一起高效地完成工作！

---
> Source: [xyfll7/xyfs_sys](https://github.com/xyfll7/xyfs_sys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
