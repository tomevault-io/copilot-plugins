## testing-tool

> 本文件为 Gemini CLI 提供关于 **Testing Tools** 浏览器扩展项目的架构说明、开发规范和技术上下文。

# Testing Tools 项目指南

本文件为 Gemini CLI 提供关于 **Testing Tools** 浏览器扩展项目的架构说明、开发规范和技术上下文。

## 1. 项目概览

- **名称**: Testing Tools
- **核心框架**: [WXT (Web Extension Toolkit)](https://wxt.dev/)
- **前端技术栈**: React 19 (Functional Components + Hooks) + TypeScript
- **UI 组件库**: Material UI (MUI) 7.x (深度定制 `sx` 属性)
- **日期处理**: dayjs (配合 timezone 和 utc 插件)
- **主要功能**: 提供时间戳转换、日期格式化等开发辅助工具。

## 2. 项目结构

```text
├── .github/              # CI/CD 工作流
├── .husky/               # Git Hooks (pre-commit linting)
├── assets/               # 静态资源 (SVG 等)
├── components/           # 复用 UI 组件
├── entrypoints/          # 浏览器扩展入口点
│   ├── background.ts     # 后台 Service Worker 逻辑
│   ├── content.ts        # 内容脚本注入逻辑
│   ├── popup/            # 扩展弹出层 (主要功能区)
│   └── options/          # 扩展选项页面
├── types/                # 全局 TypeScript 类型声明
├── utils/                # 工具类 (存储、消息通信、日期处理封装)
├── wxt.config.ts         # WXT 框架与 Manifest 配置
└── package.json          # 依赖管理与脚本
```

## 3. 开发规范与风格约定

### 3.1 UI 设计语言

- **极简主义 (Minimalist)**: 参考 Vercel 和 Apple 的设计语言。
- **MUI 定制**:
  - 严禁使用 MUI 默认的粗犷边框和深重阴影。
  - 必须通过 `sx` 属性进行深度样式定制，去除 `notchedOutline`。
  - 偏好使用 `grey.50` 背景区分层级，使用 `borderRadius: 4` (大圆角)。
  - 交互反馈：禁用波纹效果 (`disableRipple`)，移除默认阴影 (`disableElevation`)。
- **布局**: 优先使用 `Stack` 和 `Box` 进行布局，确保自上而下的操作流顺畅。

### 3.2 技术选型惯例

- **日期转换**: 必须通过 `utils/dayjs.ts` 导出的实例进行，确保时区处理一致。
- **状态管理**: 优先使用 React 原生 `useState` 和 `useMemo`。
- **存储**: 使用 `utils/chromeStorage.ts` 封装的类型安全接口。
- **通信**: 使用 `@webext-core/messaging` 进行 background 和 popup 之间的消息传递。

## 4. 关键指令

### 4.1 开发与调试

- `npm run dev`: 启动 Chrome 扩展开发模式。
- `npm run dev:firefox`: 启动 Firefox 扩展开发模式。

### 4.2 构建与检查

- `npm run build`: 构建生产版本。
- `npm run compile`: TypeScript 类型检查。
- `npm run lint`: ESLint 代码风格检查。

## 5. 权限与清单 (Manifest)

- **核心权限**: `storage`, `unlimitedStorage`, `clipboardWrite`, `scripting`, `tabs`, `debugger`, `cookies`。
- **宿主权限**: `<all_urls>` (用于在所有页面注入 content 脚本)。
- **构建细节**: 生产环境构建使用 `terser` 压缩，并强制 `ascii_only` 以确保字符兼容性。

## 6. 维护者提示

在修改 `TimestampPage.tsx` 等核心页面时，应保持**逻辑层**（基于 dayjs 的转换算法）与**渲染层**（JSX/MUI 样式）的严格分离。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LingandRX)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LingandRX)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
