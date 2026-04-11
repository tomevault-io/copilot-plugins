## uiweb-vue

> 本文件为自动化代理（如 Claude, OpenCode 等）提供在 `uiweb-vue--2.0` 存储库中高效工作的指令和规范。

# 代理指南 (AGENTS.md)

本文件为自动化代理（如 Claude, OpenCode 等）提供在 `uiweb-vue--2.0` 存储库中高效工作的指令和规范。

## 1. 项目概述
这是一个基于 **Vue 3 (Composition API)** 和 **Vite** 开发的 UI 展示/原型项目。
- **UI 库**: Arco Design (@arco-design/web-vue)
- **状态管理**: Pinia (以及部分简单的 reactive store)
- **路由**: Vue Router
- **图标**: IconPark (@icon-park/vue-next)
- **工具**: Axios, html2canvas, SVGA

## 2. 常用命令
### 构建与开发
- **启动开发服务器**: `npm run dev`
- **构建项目**: `npm run build`
- **预览构建产物**: `npm run preview`

### 部署与发布
- **部署到服务器**: `npm run deploy` (运行 `deploy.js`)
- **上传资源到 OSS**: `npm run upload-oss`
- **强制上传资源到 OSS**: `npm run upload-oss-force`

### 测试与校验
- *注意*: 当前 `package.json` 中未配置专门的测试框架（如 Vitest 或 Jest）。如需添加测试，请先与用户确认。
- **Lint**: 当前未配置专门的 Lint 命令，建议遵循 Vue 3 官方推荐风格。

## 3. 代码风格指南

### 3.1 核心原则
- **Vue 3 优先**: 始终使用 `<script setup>` 语法。
- **TypeScript 混合**: 项目中同时存在 `.js` 和 `.ts` 文件，优先使用 TypeScript。
- **别名使用**: 始终使用 `@/` 指向 `src/` 目录。

### 3.2 命名约定
- **组件**: 采用 PascalCase (如 `DeviceContainer.vue`)。如果是针对特定设备，使用连字符后缀 (如 `mine-phone.vue`, `mine-pad.vue`)。
- **目录**: 页面存放在 `src/pages/`，通用组件存放在 `src/components/`。
- **变量/函数**: 采用 camelCase。
- **常量**: 采用 UPPER_SNAKE_CASE。

### 3.3 目录结构
- `src/assets/`: 静态资源（图片、样式、SVGA 动画）。
- `src/components/`: 可复用组件。
- `src/config/`: 项目配置文件（如 `pages.json`）。
- `src/hooks/`: 自定义组合式函数 (Composables)。
- `src/pages/`: 页面组件，通常对应 `pages.json` 中的配置。
- `src/stores/`: Pinia 或 reactive 状态。
- `src/utils/`: 工具函数。

### 3.4 导入规范
- 优先从 `@/` 路径导入。
- Vue 核心 API 导入: `import { ref, computed, onMounted } from 'vue';`

### 3.5 错误处理
- 异步操作（如 Axios 请求）应包含基本的 `try-catch`。
- UI 层面的反馈优先使用 Arco Design 的 `Message` 或 `Notification` 组件。

### 3.6 动态路由与配置
- 项目使用 `src/config/pages.json` 驱动动态路由生成。
- 添加新页面时，通常需要更新 `pages.json` 并创建对应的组件文件。

## 4. 开发注意事项
- **多设备支持**: 注意代码中对 Phone 和 Pad 的区分处理（常见于组件拆分）。
- **资源路径**: 图片资源可能需要通过 OSS 托管，参考 `scripts/upload-to-oss.js`。
- **Vite 配置**: 插件配置位于 `vite.config.js`，注意自动导入插件 (`unplugin-auto-import`) 的使用。

## 5. 任务执行协议
1. 修改代码前，先使用 `Read` 工具分析相关逻辑和上下文。
2. 涉及新依赖时，检查 `package.json` 是否已有，避免重复安装。
3. 修改完成后，告知用户相关变动，并根据需要运行 `npm run build` 确保无编译错误。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qihaogithub)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qihaogithub)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
