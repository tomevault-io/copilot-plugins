## project-info

> 项目基本信息、工作流和常用命令


# 项目信息与工作流

## 🚀 快速开始
- **开发**: `pnpm dev` (H5), `pnpm dev:mp-weixin` (微信小程序)。
- **构建**: `pnpm build:h5`, `pnpm build:mp-weixin`。
- **Lint**: `pnpm lint:fix`。
- **预览**: `dist/` 目录。

## 📂 目录结构概览
- `src/pages`: 应用视图 (基于文件的路由)。
- `src/components`: 可复用的 UI 组件。
- `src/store`: Pinia 状态管理模块。
- `src/api`: Alova.js API 定义。
- `src/uni_modules`: Uni-app 模块 (包含 wot-design-uni)。

## 🤝 Git 工作流
- **提交**: 符合 Conventional Commits 规范 (`feat:`, `fix:`, `chore:`, `refactor:`)。
- **主要工具**: `commitizen` (通过 `pnpm commit` 使用)。

## 🛠️ 代码生成
- **API**: 使用 `pnpm alova-gen` 重新生成 API 定义。
- **Skills**: 使用 `.agent/skills` 中的 skill 进行脚手架生成:
    - `pinia-store-generator`
    - `uni-page-generator`
    - `alova-api-module`

---
> Source: [wot-ui/wot-starter](https://github.com/wot-ui/wot-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
