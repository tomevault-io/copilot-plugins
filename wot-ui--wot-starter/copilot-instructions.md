## tech-stack

> 核心技术栈、架构和代码规范


# 技术栈与架构

## 🏗️ 架构概览
本项目采用分层架构：
- **表现层**: `src/pages` (视图) + `src/components` (UI 逻辑)
- **状态层**: `src/store` (Pinia)
- **数据层**: `src/api` (Alova.js)

## 📦 状态管理 (Pinia)
- **库**: Pinia
- **持久化**: 使用 `src/store/persist.ts` 进行本地存储。
- **模式**: 详见 `pinia-store-generator` skill。
- **规则**: 始终使用 `defineStore` 并遵循 `use{Name}Store` 的命名规范。
- **Skill**: 使用 **`pinia-store-generator`** 快速创建新的 store。

## 🌐 API 层 (Alova.js)
- **库**: Alova.js
- **结构**:
    - `src/api/core`: 拦截器和实例配置。
    - `src/api/apiDefinitions.ts`: 自动生成的 API 定义。
- **Mock**: 支持在 `src/api/mock` 中编写 Mock 数据。
- **Skill**: 使用 **`alova-api-module`** 创建新的 API 模块和 Mock 数据。

## 🛣️ 路由
- **库**: `@wot-ui/router` (API) + `vite-plugin-uni-pages` (文件系统路由)
- **配置**: `pages.config.ts` 控制 `pages.json` 的生成。
- **导航**: 使用 `useRouter()` 进行 push/replace/back 操作。
- **Skill**: 参考 **`wot-router-usage`** 了解导航模式和守卫用法。
- **Skill**: 使用 **`uni-page-generator`** 创建带路由配置的新页面。

---
> Source: [wot-ui/wot-starter](https://github.com/wot-ui/wot-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
