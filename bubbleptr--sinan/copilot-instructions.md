## sinan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 语言要求

- 交流过程中请使用中文
- 文档使用中文撰写，存放在 docs 目录下
- 代码注释使用中文

## 协作约定

- 优先做小步、最小化改动
- UI 修改需遵循 Base UI 设计系统
- 没有明确理由时，不新增依赖

## 范围

- 主要语言：TypeScript（React/Electron）、Python（FastAPI）
- 前端：React + Vite + Tailwind + Base UI
- 后端：`sinan-core` 中的 FastAPI

## 项目概述

司南 (Sinan) 是一个 AI 驱动的移动端自动化测试平台，支持 Android 和鸿蒙 Next 双端自动化测试。采用视觉感知（VLM）替代传统控件 ID 定位，基于 AI Agent 技术实现探索式测试。

## 技术架构

**混合架构桌面应用**：
- 前端：Electron 30 + React 18 + TypeScript + Vite 5
- 后端核心：Python 3.12 + FastAPI
- 包管理：pnpm（前端）+ uv（Python）

## 常用命令

### 前端开发
```bash
pnpm dev          # 启动开发服务器
pnpm build        # 构建并打包 Electron 应用
pnpm lint         # ESLint 代码检查
pnpm preview      # 预览构建结果
```

### Python 后端 (sinan-core/)
```bash
cd sinan-core
uv sync           # 安装依赖
uv run pytest     # 运行测试
uv run ruff check # 代码检查
uv run python main.py  # 运行主程序
```

## 核心目录

- `electron/` - Electron 主进程和预加载脚本
- `src/` - React 前端源码
- `sinan-core/` - Python 后端核心模块（设备驱动、AI 模型集成）

## 备注

- UI tokens 在 `src/index.css`
- Base UI 组件样式在 `src/styles/base-ui.css`

## 架构要点

**Electron IPC 通信**：
- 主进程 (`electron/main.ts`) 管理窗口和应用生命周期
- 预加载脚本 (`electron/preload.ts`) 通过 contextBridge 暴露安全的 ipcRenderer API
- 渲染进程通过 `window.ipcRenderer` 与主进程通信

**Python 核心职责**：
- 设备驱动层（ADB/HDC 协议）
- AI 模型集成（视觉识别、OCR）
- 截图分析和操作执行

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BubblePtr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BubblePtr)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
