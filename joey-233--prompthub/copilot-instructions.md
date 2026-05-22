## prompthub

> 本项目是 Prompt Hub —— 一个 Electron 桌面应用，用于管理 AI 绘图和 LLM 提示词。

# CLAUDE.md

本项目是 Prompt Hub —— 一个 Electron 桌面应用，用于管理 AI 绘图和 LLM 提示词。

## 项目概要

- 悬浮球常驻桌面，单击展开菜单，双击打开主面板
- 主面板两个视图：提示词库（卡片网格 + 快速录入）、测试台（选词出图）
- 数据全部存本地 SQLite

## 技术栈

- Electron（electron-vite 脚手架）
- React 19 + TypeScript
- Tailwind CSS 4
- Zustand（状态管理）
- better-sqlite3（数据库）
- 出图服务：预留 adapter 接口，暂不实现

## 架构约束

- 悬浮球和主面板是两个独立的 BrowserWindow，共享主进程中的 db.ts
- 所有数据库操作在主进程，渲染进程通过 preload.ts 暴露的 IPC bridge 调用
- 悬浮球窗口：frame:false, transparent:true, alwaysOnTop:true, skipTaskbar:true
- 关闭主面板只隐藏窗口，右键悬浮球「退出」才真正退出

## 目录结构

```
electron/main.ts            主进程入口
electron/floatingBall.ts    悬浮球窗口
electron/mainWindow.ts      主面板窗口
electron/preload.ts         IPC bridge
electron/db.ts              SQLite CRUD
src/App.tsx                 主面板根组件
src/floating/FloatingBall.tsx  悬浮球 UI
src/views/Library.tsx       提示词库视图
src/views/TestBench.tsx     测试台视图
src/components/             UI 组件
src/services/ai.ts          AI 优化调用
src/services/imageGen.ts    出图 adapter（预留）
src/stores/store.ts         Zustand store
```

## 数据模型

prompts 表：id, title, content, notes, tags(JSON), params(JSON), is_favorite, last_used_at, last_generated_at, use_count, created_at, updated_at
generations 表：id, prompt_id, provider_id, status, prompt_title_snapshot, prompt_snapshot, image_data, params(JSON), created_at

- 提示词分类合并到 tags：`'绘图'` 和 `'LLM'` 是两个保留 tag 值（src/shared/types.ts 的 IMAGE_TAG / LLM_TAG），UI 把它们当作首要类型 chip 来显示和筛选；TestBench 只列含 `'绘图'` tag 的提示词
- 老库会在启动时自动迁移：`type` 列里的 'image'/'llm' 折进 tags 后 ALTER TABLE DROP COLUMN type

## 开发命令

```bash
npm run dev          # 开发模式
npm run build        # 构建
npm run preview      # 预览构建产物
```

## 注意事项

- 全界面中文
- better-sqlite3 安装时需要 node-gyp 编译环境（Windows 需要 Visual Studio Build Tools）
- Windows 上若出现 transparent 窗口闪烁或黑块，可在 main.ts 启动前加回 `app.disableHardwareAcceleration()`（默认不加——禁用硬件加速会让全 app 走软件渲染，明显卡顿）
- 悬浮球拖拽用 -webkit-app-region: drag，菜单区域用 no-drag

---
> Source: [Joey-233/PromptHub](https://github.com/Joey-233/PromptHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
