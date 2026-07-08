## codex-cua-mcp

> 本文档为 Claude Code（claude.ai/code）在此仓库中工作时提供指导。

# CLAUDE.md

本文档为 Claude Code（claude.ai/code）在此仓库中工作时提供指导。

## 项目概述

Codex CUA MCP 将 Codex 的 `codex-computer-use.exe` 封装为标准 MCP 服务器，使任何支持 MCP 的智能体（Claude Code、Cursor、Windsurf 等）能够通过截图、点击、打字和键盘输入来控制 Windows 桌面应用程序。

## 命令

```bash
npm run build          # 将 TypeScript 编译到 dist/
npm start              # 启动 MCP 服务器（需要 dist/ 已存在）
```

## 架构

```
智能体（Claude Code、Cursor 等）
  ↓ MCP 协议（stdio）
src/index.ts            # MCP 服务器入口，stdio 传输层，读取 clientInfo
  ↓
src/server.ts           # CodexCuaServer - 定义 13 个工具，懒启动 exe
  ↓
src/agent-name.ts       # 解析调用方显示名（env > 映射表 > Title Case > 兜底）
src/runtime-config.ts   # 为每个 Agent 渲染独立 config 到 config-runtime/<slug>/
  ↓
src/transport/sky.ts    # SkyTransport - 启动 exe，通过 stdin/stdout 进行 JSON-RPC 通信
  ↓
bin/codex-computer-use.exe  # 原生 Windows 二进制文件（SendInput、UI Automation）
```

**关键流程**：MCP 握手完成 → 首次工具调用时 `server.ts` 解析 Agent 名、渲染运行时 config、懒启动 exe → `sky.ts` 发送 JSON-RPC → exe 在 Windows 上执行 → 返回响应。多个 Agent 各自启动独立 exe 进程与 `CODEX_HOME`，互不冲突。

## 通信协议

exe 通过 stdin/stdout 使用换行分隔的 JSON-RPC 协议：
- 请求：`{"id": N, "method": "...", "params": {...}}`
- 成功：`{"id": N, "ok": true, "result": {...}}`
- 需要审批：`{"id": N, "ok": false, "approvalRequest": {"app": "...", "displayName": "...", "riskLevel": "low"}}`
- 审批响应：重新发送原始请求并附带 `meta: {"x-oai-cua-approved-app": "<app>"}`

## 重要实现细节

- **懒启动**：exe 在首次工具调用时才 spawn（`CodexCuaServer.ensureStarted()`），握手阶段不启动。`index.ts` 通过 `getClientInfo` 回调把 `server.getClientVersion()` 传入，供名字解析使用。
- **动态 Agent 名**：`agent-name.ts` 按优先级解析调用方显示名——`CUA_AGENT_NAME` 环境变量 > MCP `clientInfo.name` 映射表 > Title Case 兜底 > `"AI Agent"`。名字写入桌面操作浮层文案。
- **审批处理**：每个操作（launch_app、get_window_state、click、type_text 等）在首次使用某个应用时会触发审批请求。`SkyTransport` 默认自动审批，并重新发送带有审批元数据的请求。
- **窗口状态要求**：输入操作（click、type、press_key、scroll、drag）需要先对目标窗口调用 `get_window_state`。
- **配置隔离**：`runtime-config.ts` 把 `config/computer-use/config.json` 模板（含 `{{AGENT_NAME}}` 占位符）渲染到 `config-runtime/<slug>/computer-use/config.json`，`CODEX_HOME` 指向该目录。每个 Agent 独立目录，多 Agent 互不冲突。
- **exe 搜索顺序**：`SKY_EXE_PATH` 环境变量 → 捆绑的 `bin/codex-computer-use.exe` → Codex 安装目录。

## 配置

`config/computer-use/config.json` 是模板，控制 Computer Use 会话期间显示的弹出文本。运行时按 Agent 名渲染到 `config-runtime/<slug>/`（已 gitignore）。

---
> Source: [RS-Nocsi/codex-cua-mcp](https://github.com/RS-Nocsi/codex-cua-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
