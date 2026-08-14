## jsonhandle

> 本文件用于指导在本仓库中运行的智能体如何协作与执行任务。

# AGENTS.md

本文件用于指导在本仓库中运行的智能体如何协作与执行任务。

## 语言

- 所有说明与回复使用中文。

## 工作方式

- 优先阅读仓库内已有文档（如 `README.md`、`CLAUDE.md`）以理解项目背景。
- 修改代码前先定位相关文件与上下文，避免盲改。
- 变更保持最小化、可追踪；不做无关重构。
- 涉及运行命令或写文件时，说明目的并确保可复现。
- UI 改动需保证响应式，适配不同窗口尺寸。

## 项目概览

- Safari 浏览器扩展：自动检测并格式化页面中的 JSON，提供树形折叠、高亮、搜索与复制/下载等功能。
- 主要逻辑在 `jsonHandle Extension/Resources/content.js`。
- 原生宿主为 macOS app（Swift + SafariServices）。

## 目录结构

- `jsonHandle/`：macOS 宿主应用（Swift/Cocoa）
- `jsonHandle Extension/`：Safari Web Extension
- `jsonHandle Extension/Resources/`：扩展前端资源（`content.js`、`background.js`、`popup.*`、`manifest.json`）
- `jsonHandleTests/`：单元测试（占位为主）
- `jsonHandleUITests/`：UI 测试（占位为主）

## 构建与运行

```bash
# 使用 Xcode 打开项目
open jsonHandle.xcodeproj

# 命令行构建
xcodebuild -project jsonHandle.xcodeproj -scheme jsonHandle -configuration Debug build
```

## 开发注意事项

- 扩展权限与配置在 `manifest.json` 和 `Info.plist` 中。
- JS 代码包含大量中文注释，改动时注意保持一致与可读性。
- 兼容要求：macOS 11+、Safari 14+。
- 调试：Safari Develop 菜单 > Show Extension Console。

---
> Source: [jojo-jie/jsonHandle](https://github.com/jojo-jie/jsonHandle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
