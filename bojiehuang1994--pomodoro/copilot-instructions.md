## pomodoro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

一个基于 Electron 的桌面番茄钟应用。计时器支持 25 分钟专注 + 5 分钟休息循环，圆形进度环显示，系统托盘常驻，通知提醒，本地持久化每日统计。

## 命令

```bash
npm start          # 启动 Electron 应用
npm run dist       # 打包发布（electron-builder）
```

## 架构

```
main.js              # Electron 主进程 — 窗口、托盘、IPC、统计持久化
preload.js           # contextBridge 暴露 electronAPI（stats:get, stats:record, notify）
renderer/
  index.html         # 界面结构
  style.css          # 暗色主题样式，进度环 SVG，按钮
  renderer.js        # 前端逻辑 — 计时状态机、模式切换、进度环更新、统计渲染
```

### 关键设计

- **统计持久化**：主进程将每日专注数据以 JSON 格式存入 `app.getPath('userData')/stats.json`，结构为 `{ "YYYY-MM-DD": { focusCount, focusMinutes } }`。渲染进程通过 IPC 的 `stats:get` / `stats:record` 读写。
- **托盘行为**：关闭窗口时默认隐藏到托盘而非退出（`mainWindow.on('close')` + `preventDefault`），通过托盘菜单真正退出。
- **上下文隔离**：`contextIsolation: true`，`nodeIntegration: false`，所有 Node.js 能力通过 preload 桥接暴露。
- **计时完成后自动切换**：专注完成 → 自动进入休息模式并开始计时；休息完成 → 切换回专注模式进入空闲。
- **CSP**：`default-src 'self'`，脚本样式严格限制，无外部依赖。

---
> Source: [BojieHuang1994/Pomodoro](https://github.com/BojieHuang1994/Pomodoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
