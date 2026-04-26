## yewtab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

yew-tab 是一个基于 React + Vite 构建的浏览器扩展（Manifest V3），为 Edge/Chrome 提供自定义新标签页，以卡片形式展示和管理书签。

## 常用命令

```bash
# 安装依赖
pnpm install

# 开发模式（支持热重载）
pnpm dev

# 构建生产版本
pnpm build

# 代码检查
pnpm lint

# 预览构建产物
pnpm preview
```

**加载扩展到浏览器：**
1. 运行 `pnpm build`
2. 打开 `edge://extensions` 或 `chrome://extensions`
3. 启用"开发人员模式"
4. 点击"加载解压缩的扩展"，选择 `dist/` 目录
5. 代码修改后重新运行 `pnpm build` 并点击扩展的刷新按钮

**热重载开发：**
- 运行 `pnpm dev` 后加载扩展，大部分前端修改会自动更新
- 修改 manifest、background script 或添加新文件需要手动刷新扩展

## 架构设计

### 前后端分离 + 消息驱动架构

```
┌─────────────────────────────────────────┐
│  Frontend: React 新标签页 (newtab/)      │
│  └─ 通过 chrome.runtime.sendMessage     │
│     与后台通信                           │
├─────────────────────────────────────────┤
│  Backend: Service Worker (background/)   │
│  └─ 处理书签 CRUD、缓存同步、消息路由     │
├─────────────────────────────────────────┤
│  Chrome APIs                            │
│  └─ bookmarks / storage / runtime       │
└─────────────────────────────────────────┘
```

### 核心模块职责

- **src/newtab/** - React 前端应用，包含 UI 组件和应用入口
- **src/background/** - Service Worker 后台脚本，消息路由与书签监听
- **src/lib/** - 核心业务逻辑
  - `bookmarks/` - Chrome Bookmarks API 封装
  - `messaging/` - 前后端消息通信封装
  - `storage/` - 书签快照与布局状态持久化
- **src/shared/** - 前后端共享的类型定义和常量

### 数据流

1. **书签加载**: 前端 → messaging → background → chrome.bookmarks.getTree() → 返回树结构
2. **书签变更**: 前端操作 → messaging → background → chrome.bookmarks API → 更新缓存 → 广播变更消息
3. **离线降级**: 后台自动维护书签快照，前端获取失败时降级到本地缓存

### 关键架构特性

1. **自定义 Hooks 系统**: 应用通过多个自定义 Hooks 管理状态
   - `useBookmarks` - 书签数据加载与监听
   - `useLayoutState` - 布局与导航状态持久化
   - `useFolderNavigation` - 文件夹导航与展开/收起逻辑
   - `useSortableOrder` - 拖拽排序逻辑（基于 @dnd-kit）
   - `useContextMenu` - 右键菜单状态管理

2. **拖拽排序**: 使用 @dnd-kit 库实现书签卡片的拖拽重排
   - 前端 UI 由 `@dnd-kit/core` 和 `@dnd-kit/sortable` 处理
   - 实际顺序变更通过 `reorderBookmarkChildren` 消息提交到 background

3. **状态持久化**: 使用 chrome.storage.local 存储
   - 书签快照（`bookmarks_snapshot`）- 离线降级使用
   - 布局状态（`layout_state`）- 固定书签、启动文件夹、展开状态等
   - 搜索设置（`search_settings`）
   - 背景设置（`background_settings`）- 支持明暗主题独立配置

4. **版本信息注入**: vite.config.ts 在构建时注入 Git 版本信息
   - `__APP_VERSION__`: git tag 或 commit hash
   - `__GIT_COMMIT__`: 短 commit hash

## 代码规范

- **文件命名**: kebab-case（如 `bookmark-card.tsx`）
- **组件风格**: 函数式组件 + Hooks
- **提交规范**: 约定式提交，使用中文（如 `feat: 新增搜索框`、`fix: 修复拖拽排序`）
- **语言**: 代码注释、文档、提交信息均使用中文
- **路径别名**: 使用 `@/` 指向 `src/` 目录（已配置在 vite.config.ts）

## 关键文件

- [src/manifest.ts](src/manifest.ts) - 扩展 Manifest V3 配置
- [src/newtab/app.tsx](src/newtab/app.tsx) - 主应用组件，组装所有 Hooks 与 UI
- [src/background/index.ts](src/background/index.ts) - 后台服务入口，消息路由与书签监听
- [src/shared/types/index.ts](src/shared/types/index.ts) - 核心类型定义
- [src/shared/constants/index.ts](src/shared/constants/index.ts) - 消息类型常量与存储键
- [vite.config.ts](vite.config.ts) - Vite 配置，包含 @crxjs/vite-plugin 热重载设置

## 开发注意事项

1. **修改 Chrome API 调用时**: 更新 [src/lib/bookmarks/index.ts](src/lib/bookmarks/index.ts) 或 [src/lib/messaging/index.ts](src/lib/messaging/index.ts)
2. **添加新的持久化状态**: 在 [src/lib/storage/index.ts](src/lib/storage/index.ts) 中添加读写函数，并在 [src/shared/constants/index.ts](src/shared/constants/index.ts) 中定义 STORAGE_KEYS
3. **添加前后端消息类型**: 在 MESSAGE_TYPES 常量中定义，并在 background/index.ts 的消息监听器中处理
4. **调试 Service Worker**: 在 `edge://extensions` 中点击扩展的"Service Worker"链接查看后台日志
5. **UI 组件库**: 使用 Tailwind CSS v4 + 自定义 UI 组件（位于 [src/components/ui/](src/components/ui/)）
6. **动画**: 使用 framer-motion 库实现，参考现有组件（如 [src/newtab/components/bookmark-card.tsx](src/newtab/components/bookmark-card.tsx)）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YewFence) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
