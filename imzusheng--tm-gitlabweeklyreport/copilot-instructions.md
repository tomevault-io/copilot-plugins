## tm-gitlabweeklyreport

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

始终使用中文回复

## 项目概述

这是一个基于 DeepSeek AI 的 GitLab 工作周报自动生成工具，支持双模式部署：

- **Tampermonkey 用户脚本模式**：在 GitLab 页面直接注入界面
- **Web 应用模式**：独立的 Web 应用

### 核心功能

- AI 智能生成周报（基于 DeepSeek AI）
- GitLab 事件数据获取和筛选
- 响应式设计（支持桌面和移动设备）
- 主题系统（深色/浅色/跟随系统）
- 数据持久化和配置管理

## 开发命令

### 核心开发命令

- `npm run dev` - 启动开发服务器 (localhost:3000)
- `npm run build` - 构建所有版本（Web 应用 + 油猴脚本）
- `npm run build:userscript` - 仅构建油猴脚本版本
- `npm run build:web` - 仅构建 Web 应用版本

### 代码质量检查

- `npm run check` - 运行所有检查（lint + stylelint + type-check）
- `npm run lint` - ESLint 检查
- `npm run type-check` - TypeScript 类型检查
- `npm run stylelint` - Less 样式检查
- `npm run format` - Prettier 代码格式化

### 发布相关

- `npm run release` - 语义化发布
- `npm run release:dry-run` - 预览发布内容
- `npm run commitlint` - 提交信息检查

## 项目架构

### 核心技术栈

- **React 18** + **TypeScript** + **Vite**
- **CSS Modules** + **Less** 预处理器
- **useReducer** + **Context** 状态管理
- 双模式构建支持（Web 应用 + 油猴脚本）
- **Shadow DOM** 样式隔离（油猴脚本模式）

### 目录结构

```
src/
├── components/          # React 组件
│   ├── AIPanel/         # AI 周报生成面板
│   ├── MainPanel/       # 主面板（事件列表）
│   ├── SettingsPanel/   # 设置面板
│   └── ...              # 其他UI组件
├── hooks/               # 自定义 React Hooks
│   ├── useAppState.ts   # 主状态管理 Hook
│   ├── useEventManagement.ts  # 事件管理
│   ├── useThemeManager.ts     # 主题管理
│   └── ...              # 其他业务 Hooks
├── services/            # API 服务层
│   ├── gitlab-api.ts    # GitLab API 封装
│   └── deepseek-api.ts  # DeepSeek AI API 封装
├── stores/              # Zustand 状态管理
│   └── app-store.ts     # 全局状态 Store
├── types/               # TypeScript 类型定义
├── utils/               # 工具函数
├── constants/           # 常量配置（包含动态配置）
├── styles/              # 全局样式
│   ├── css-variables.ts # CSS 变量定义
│   ├── base.less        # 基础样式
│   └── ...              # 其他样式文件
├── main.tsx            # Web 应用入口
└── userscript.ts       # 油猴脚本入口
```

### 状态管理

- 使用 **Zustand** 进行状态管理（从 `src/stores/app-store.ts` 导出）
- 支持配置持久化（localStorage/GM_setValue）
- 分离关注点：配置状态、UI状态、事件数据状态
- 提供选择器 Hooks：`useConfig`、`useTheme`、`useEvents` 等
- 提供操作 Hooks：`useAppActions` 用于状态更新

### 构建系统

- **双模式构建**：Vite 配置自动处理 Web 和 Userscript 两种模式
- **CSS注入**：油猴脚本模式使用自定义 CSS 注入到 Shadow DOM
- **路径别名**：`@/` 映射到 `src/` 目录
- **外部依赖**：油猴脚本模式外部化 React/ReactDOM
- **Shadow DOM 支持**：样式完全隔离，防止与宿主页面冲突

## 开发注意事项

### 类型安全

- 启用 TypeScript 严格模式
- 优先使用具体类型而非 `any`
- 所有组件和函数都需要类型定义

### 样式规范

- 使用 CSS Modules + Less
- 命名约定：`camelCase` 局部类名
- 避免使用 `styles["kebab-case"]` 语法

### 环境适配

- 代码需同时支持浏览器环境和油猴脚本环境
- 使用 `isUserscriptEnvironment()` 检测环境
- 开发模式下使用代理解决 CORS 问题

### Shadow DOM 开发

- 油猴脚本模式使用 Shadow DOM 完全隔离样式
- CSS 通过全局变量 `window.__GITLAB_WEEKLY_REPORT_STYLES__` 注入
- 触发按钮在主文档中，主应用在 Shadow DOM 内
- 配置项存储在 `src/constants/index.ts` 的 `USERSCRIPT_CONFIG` 中

### API 调用

- GitLab API 使用 `createGitLabApiService()`
- DeepSeek AI API 使用 `DeepSeekApiService`
- 统一错误处理和重试机制
- 支持请求取消（AbortSignal）

### 版本管理

- 采用语义化版本控制
- 使用 Conventional Commits 规范
- 版本号自动同步到所有相关文件
- ~~自动生成 CHANGELOG~~（功能已移除）
- 版本号从 `package.json` 统一管理，通过构建脚本同步到油猴脚本头部

## 常见任务

### 添加新组件

1. 在 `src/components/` 下创建组件目录
2. 创建 `index.tsx` 和 `index.module.less` 文件
3. 使用 TypeScript 定义 Props 接口
4. 遵循现有的组件结构和命名约定

### 修改 API 服务

1. 编辑 `src/services/` 下的相应服务文件
2. 更新 `src/types/` 中的类型定义
3. 确保错误处理和环境适配正确

### 测试构建

```bash
# 测试所有模式构建
npm run build

# 验证油猴脚本
# 检查 dist/userscript/gitlab-weekly-report.user.js

# 验证 Web 应用
# 检查 dist/web/ 目录
```

### 代码提交

遵循 Conventional Commits：

- `feat:` 新功能
- `fix:` 修复 bug
- `refactor:` 代码重构
- `style:` 样式调整
- `docs:` 文档更新

运行提交前检查：

```bash
npm run check
npm run format
```

### 重要开发规范

- **UI/UX 设计**：遵循 macOS 设计语言，支持深色/浅色/系统主题
- **TypeScript**：启用严格模式，优先使用接口而非类型别名
- **组件规范**：使用函数组件和 Hooks，Props 接口命名格式为 `{ComponentName}Props`
- **导入规范**：使用 `@/` 别名进行绝对路径导入，按特定顺序组织导入语句
- **样式规范**：使用 Less 预处理器，组件样式文件命名为 `index.less`
- **性能优化**：合理使用 `useMemo`、`useCallback`，大型组件使用 `React.memo`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imzusheng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
