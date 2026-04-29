## yxaicode

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## 项目概述

@yxai/code（意心Code）是 Claude Code 的 Web 可视化界面封装。基于 Node.js + Express + WebSocket + 原生 HTML/CSS/JS，无构建工具、无前端框架。核心代码约 4600 行，分布在 `server.js`（后端）和 `public/app.js`（前端）两个主文件中。

## 常用命令

```bash
npm install          # 安装依赖
npm start            # 启动服务（等同 node server.js），默认端口 6060
npm run dev          # 同上
yxai --port 8080     # 指定端口启动
npm publish --access public  # 发布到 npm
```

本项目无测试、无 lint、无构建步骤。

## 架构

### 三层结构

```
前端 (public/)  ←WebSocket→  后端 (server.js)  → Claude Agent SDK (@anthropic-ai/claude-agent-sdk)
```

### 后端 server.js

单文件 Express 服务，职责：
- 静态文件服务（`public/` 目录）
- REST API：版本查询、模型列表（代理 yxai.chat）、项目/会话管理、文件浏览
- WebSocket 服务：接收用户指令 → 调用 `query()` (Claude Agent SDK) → 流式推送结果
- 权限审批：通过 `pendingApprovals` Map + Promise 实现异步工具审批流程
- 会话管理：`activeSessions` Map 跟踪活跃 Claude 会话，支持中止

关键函数：`runQuery()`（执行 Claude SDK 查询）、`waitForApproval()`（权限审批）、`loadMcpConfig()`（加载 MCP 配置）

### 前端 public/app.js

原生 JS 单页应用，无框架。全局状态变量管理（`ws`, `sessionId`, `isStreaming`, `selectedModel` 等）。

核心功能：
- WebSocket 客户端，处理消息流式渲染
- 斜杠命令系统：`/clear`, `/new`, `/rewind`, `/help`, `/model`, `/init`
- 文件 @ 提及自动补全
- 权限审批 UI（记住权限规则）
- 侧边栏会话管理（项目/会话树）
- 文件浏览面板 + 目录选择器
- 深色/浅色主题切换（CSS 变量驱动，`data-theme` 属性）

### 前端 public/styles.css

CSS 变量驱动的主题系统。深色主题：深黑背景 `#0a0a0a` + 橙色主题色 `#f97316`（Claude 风格）。浅色主题通过 `[data-theme="light"]` 选择器覆盖。

### WebSocket 消息协议

客户端 → 服务端：`claude-command`（发送指令）、`permission-response`（权限回复）、`abort-session`（中止会话）

服务端 → 客户端：`session-created`、`claude-response`（流式内容）、`claude-complete`、`claude-error`、`permission-request`、`token-usage`

### REST API 端点

- `GET /api/version` - 版本号
- `GET /api/models` - 模型列表（代理 yxai.chat）
- `GET /api/projects` - Claude 项目列表
- `GET /api/projects/:name/sessions/:id/messages` - 会话消息
- `GET /api/browse` - 目录浏览
- `GET /api/files` - 文件树（深度 5）
- `GET /api/files-flat` - 扁平文件列表（用于 @ 提及）
- `GET /api/file` - 读取单个文件（最大 500KB）

## 设计规范

详见 `prompt/DESIGN_SYSTEM.md`。主题色 `#f97316`，背景 `#0a0a0a`，字体 Inter，间距 4-32px，动画 150-300ms。

## 开发注意事项

- ES Modules（`"type": "module"`），使用 `import` 而非 `require`
- Node.js >= 18.0.0（推荐 >= 22.0.0）
- 前端第三方库以预构建文件形式放在 `public/vendor/`（marked、highlight.js）
- 发布到 npm 时仅包含 `server.js` 和 `public/` 目录
- 项目语言为中文，注释和 UI 文案使用中文

---
> Source: [AIDotNet/yxaicode](https://github.com/AIDotNet/yxaicode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
