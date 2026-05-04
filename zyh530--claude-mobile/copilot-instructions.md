## claude-mobile

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 启动命令

要求 Node.js 20+。依赖：`@anthropic-ai/claude-agent-sdk`。

```bash
# 启动云端中继服务（默认监听 http://127.0.0.1:3456）
npm run cloud

# 启动开发机 Agent（需指定云端地址和项目根目录）
AGENT_CLOUD_URL=ws://127.0.0.1:3456/ws \
AGENT_PROJECT_ROOT=/absolute/path/to/project \
npm run agent
```

环境变量参考 `.env.example`。运行时状态持久化在 `.runtime/cloud-state.json` 和 `.runtime/agent-state.json`（自动创建，不提交）。

健康检查：`GET http://127.0.0.1:3456/api/health`

## 架构概览

这是一个**三端系统**，允许用户通过网页远程控制开发机上运行的 Claude Agent：

```
Web 前端 (浏览器)
    ↕ WebSocket (/ws)
云端中继 (apps/cloud/server.js)
    ↕ WebSocket（开发机主动连出）
开发机 Agent (apps/agent/index.js)
```

**apps/cloud/server.js** — 云端中继，单文件，职责：
- HTTP 静态文件服务（`apps/cloud/public/`）
- WebSocket 升级与路由（区分 `web` 和 `agent` 两种客户端）
- 设备注册/在线状态维护（`agentClients` Map）
- 会话状态机管理与事件持久化
- 将 Web 指令路由到对应 Agent，将 Agent 事件广播给所有 Web 客户端

**apps/agent/index.js** — 开发机常驻 Agent，单文件，职责：
- 主动连接云端 WebSocket，自动重连（1.5s 退避）
- 发送心跳（每 10s）
- 执行会话创建/恢复/prompt/审批响应
- 使用 **Claude Agent SDK** 驱动 `query()` / `resume`，流式输出映射为项目事件协议
- 高风险操作在 prompt 层拦截，触发审批流

**packages/shared/** — 零依赖共享库：
- `protocol.js`：`SESSION_STATUS` 枚举、事件工厂、ID 生成、文本摘要、工具函数
- `json-store.js`：带防抖写入（150ms）的 JSON 文件持久化
- `ws.js`：纯 Node.js 手写 WebSocket 实现，支持服务端（`acceptWebSocket`）和客户端（`connectWebSocket`），客户端自动检测浏览器原生 `WebSocket`

## 消息协议

所有 WebSocket 消息格式：`{ type: string, payload: object }`

**Web → Cloud → Agent**：`session.create`、`session.resume`、`claude.prompt`、`approval.respond`

**Agent → Cloud → Web（通过 `agent.event` 包装）**：`session.created`、`session.state`、`claude.init`、`claude.delta`、`terminal.output`、`approval.request`、`approval.resolved`、`session.error`

Cloud 向 Web 广播 `snapshot` 消息（含完整设备列表和会话列表），Web 连接后立即收到当前快照。

## 会话状态机

`created → booting → running → waiting_user ⇌ running → completed / failed / killed`

## 关键设计约束

- **Claude API key 只放开发机**：云端和 Web 侧永远不接触 `ANTHROPIC_API_KEY`。
- **开发机只主动连出**：Agent 向 Cloud 建立出站 WebSocket，不暴露公网命令执行入口。
- **手写 WebSocket**：`ws.js` 是零依赖手写实现，`json-store.js` 是手写持久化。
- `claude.delta` 事件在 Cloud 侧会自动合并（同 turnId + 同 role 的连续增量追加到同一条事件），事件列表最多保留 120 条（`clampEvents`）。
- Agent SDK 配置：`permissionMode: "bypassPermissions"`，`allowedTools: ["Bash", "Read", "Edit", "Write", "Glob", "Grep", "Skill"]`，可通过 `CLAUDE_MODEL` 和 `CLAUDE_MAX_TURNS` 环境变量调整。

---
> Source: [zyh530/claude-mobile](https://github.com/zyh530/claude-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
