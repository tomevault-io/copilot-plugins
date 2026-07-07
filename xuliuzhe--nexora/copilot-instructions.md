## nexora

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Communication Language

**IMPORTANT**: Always respond in Chinese (中文) when working in this repository.

## Project Overview

**Nexora (灵枢) v3** — 个人 AI 工作站 Web 服务。多 Agent 协作，通过频道（channel）对话驱动，支持飞书集成。Agent 基于 Claude Agent SDK 执行，工具直接调用 Claude Code 内置工具。

Bun monorepo，三个包：`packages/server`（Hono）、`packages/web`（React 19 + Vite）、`packages/shared`（Zod schemas）。

纯文件存储，无数据库。JSONL 存频道/任务，Markdown 存记忆，YAML 存 Agent 定义。

## Development Commands

```bash
bun run dev          # 启动后端（Hono，端口 3000，--watch 热重载）
bun run dev:web      # 启动前端（Vite，端口 5173，proxy → 3000）
bun run test         # 运行所有测试（bun test）
bun run typecheck    # TypeScript 严格检查（三个 tsconfig）
bun run lint         # Biome lint + format 检查
bun run lint:fix     # 自动修复
bun run check        # 完整质量门禁：lint + typecheck + test
```

运行单个测试：
```bash
cd packages/server && bun test src/services/dispatcher.test.ts
```

LLM 配置：`.nexora/settings.json`（从 `.nexora/settings.json.example` 复制）。三个模型层级：`cheap` / `standard` / `expensive`，对应环境变量 `NEXORA_CHEAP_MODEL` / `NEXORA_STANDARD_MODEL` / `NEXORA_EXPENSIVE_MODEL`。`ANTHROPIC_API_KEY` 作为 API Key 回退。`CLAUDE_CODE_PATH` 指定 Claude Code 可执行路径（默认 `/opt/homebrew/bin/claude`）。

## Architecture

### 核心流程

用户消息 → ChannelService → `dispatchMessage()` 路由 → `executeAgentCore()` → Claude Agent SDK → 回复写入频道 → WebSocket 广播前端。

消息路由优先级（`services/dispatcher.ts`）：
1. @mention 中的有效成员 → `rule="mention"`
2. DM 频道 → 固定目标 dmAgent
3. 无匹配 → `rule="infer"`（暂未实现 LLM 路由）

### Claude Agent SDK 执行层

每次 Agent 执行调用 `services/sdkAgent.ts` → `runAgentQuery()`，内部用 `@anthropic-ai/claude-agent-sdk` 的 `query()` 函数。`permissionMode` 固定为 `bypassPermissions`，工具为 Claude Code 内置工具（`Read`, `Edit`, `Write`, `Bash`, `Glob`, `Grep`）。`cwd` 设为频道绑定的 `projectDir`。

`maxTurns`：普通聊天 5，委托任务 20，Task 模式 30。

### Agent 定义

Agent 通过目录结构定义，无继承：

```
agents/<dirName>/
  agent.yaml    # name/displayName/description/model/tools/skills
  AGENT.md      # 角色指令（注入 system prompt 的 ## Instructions 段）
  memory/
    MEMORY.md   # Agent 个人记忆（§ 分隔条目，2200 字符上限）
```

`agent.yaml` 核心字段：`name`（展示用）、`displayName`（中文别名）、`description`、`model`（`cheap`/`standard`/`expensive`）、`tools`（Claude Code 工具名数组，可选）、`skills`（技能名数组）。

### 任务编排（Task 模式）

用户发送带 `asTask=true` 的消息或消息包含链式关键词（然后/之后/完成后）时，触发 `TaskPlanner` → 用 LLM 分解为多个 `Task` → `TaskOrchestrator` 按依赖顺序触发 Agent 执行 → `TaskStateMachine` 管理状态转移：

```
todo → executing → verifying → done
                 ↓           ↓
              blocked      fixing → verifying
                           ↓
                        escalated → todo
```

### 委托系统（Delegation）

Agent 在回复中嵌入委托指令（`services/delegateTool.ts` 解析）→ `ChannelService.executeDelegation()` 递归触发目标 Agent → 最大深度 5 层（`MAX_DELEGATE_DEPTH`）。
@mention 链式调用作为兼容机制保留（最大 3 层，`MAX_CHAIN_DEPTH`）。

### 文件存储层

所有持久化通过 `packages/server/src/storage/`：
- `jsonl.ts` + `lock.ts`：`appendJsonl()`（带文件锁）+ `readJsonl()`
- `channel.ts`：`ChannelStore` — 每个频道一个 JSONL 文件，第一行 `_meta`
- `task.ts`：`TaskStore` — 任务按频道存储
- `paths.ts`：懒求值路径函数，读 `NEXORA_DATA_DIR`（默认 `./data`）

数据目录：`data/channels/`、`data/tasks/`、`data/memory/`（全局 `USER.md`）、`data/cost/`、`data/skills/`。

### 记忆系统

`services/memory/store.ts` 中 `MemoryStore`，维护两个文件：
- `MEMORY.md`（per-agent）：Agent 个人笔记，2200 字符上限
- `USER.md`（全局共享，`data/memory/USER.md`）：用户偏好，1375 字符上限

条目用 `\n§\n` 分隔。**冻结快照模式**：`load()` 时捕获系统提示注入内容，会话中间的写入操作不改变当前 system prompt（下次会话生效）。内置安全扫描，拦截提示注入和敏感信息渗漏模式。

### Bootstrap 初始化

`packages/server/src/bootstrap.ts`：加载 `.nexora/settings.json` → 构建三层模型配置 → 加载 Agent 目录 → 创建 `ChannelStore` + `TaskStore` + `HandoffTracker` + `CostTracker` → 创建 `ChannelService` → 创建 `TaskPlanner` + `TaskOrchestrator` 并注入 ChannelService → 确保默认 `#all` 频道。

`findDir()` 向上搜索父目录，解决 `bun run --filter` 从子包目录执行时路径偏移问题。

### System Prompt 构建

`services/agentRunner.ts` 中 `buildSystemPrompt()` 按固定顺序拼接：Role → Output Format → Collaboration → Channel Members → Delegation Depth Warning → Instructions（AGENT.md）→ Project Doc（PROJECT.md/CLAUDE.md）→ Memory → User Profile → Current Task → Handoff Context → Skills → Session Tasks（TodoStore）。

### 飞书集成

频道可绑定飞书群（`feishu.chatId`）。Agent 回复后异步推送飞书消息（`feishu/index.ts`）。飞书 webhook 接收消息写入对应频道。

## Coding Standards

### Hard Limits（Biome 强制）

- 文件 ≤ 300 行，函数 ≤ 50 行，认知复杂度 ≤ 15，参数 ≤ 4 个
- `biome.json`：lineWidth 100，indentWidth 2，space 缩进

### Design Principles

- **单一职责**：`jsonl.ts` 只管读写，`dispatcher.ts` 只管路由规则
- **开闭原则**：新 Agent 写目录+YAML，新技能写 Markdown，不改引擎代码
- **依赖反转**：`ChannelService` 接收 deps 对象，不直接构造依赖
- **组合优于继承**：Agent 能力通过 `tools` 数组 + `skills` 数组声明，无 BaseAgent 继承链
- **YAGNI**：不为假设需求建抽象

### 添加新 Agent

1. 创建 `agents/<name>/` 目录
2. 写 `agent.yaml`（必填：`name`、`description`、`model`、`tools`）
3. 写 `AGENT.md`（角色定义和行为指令）
4. 重启服务自动加载（`bootstrap` 调用 `loadAllAgents()`）

### 测试

- TDD：先写失败测试 → 实现 → 确认通过 → 提交
- 文件操作测试用 `mkdtemp` 临时目录，`afterAll` 清理
- 每个测试文件独立，不共享全局状态
- 需要 LLM 的测试用 mock（见 `services/agentRunner.test.ts`）

### 关键类型

```typescript
// Agent 配置（shared/src/schemas.ts → AgentConfigSchema）
interface AgentConfig { name; displayName?; description; skills: string[]; model: "cheap"|"standard"|"expensive"; tools?: string[] }

// 频道元数据（shared/src/schemas.ts → ChannelMetaSchema）
interface ChannelMeta { id; name; type: "channel"|"dm"; members: string[]; dmAgent?; projectDir?; feishu?; createdAt; archived }

// 消息（shared/src/schemas.ts → MessageSchema）
interface Message { id; channelId; threadId?; author: {type:"user"|"agent"|"system"; name; displayName?}; content; messageType: "text"|"handoff"|"summary"|"system"|"delegate"|"delegate_result"; mentions: string[]; metadata?; createdAt }

// 任务（shared/src/schemas.ts → TaskSchema）
interface Task { id; channelId; title; description; acceptanceCriteria: string[]; status: "todo"|"executing"|"verifying"|"fixing"|"done"|"blocked"|"escalated"; assignee?; verifier?; dependsOn: string[]; createdAt; updatedAt }

// Agent SDK 查询（services/sdkAgent.ts）
interface AgentQueryResult { text: string; sessionId?; usage?: { inputTokens; outputTokens; totalCostUsd; model } }
```

## Browser Testing

**不要使用 Playwright**。使用 Chrome DevTools Protocol (CDP) 直接连接浏览器进行截图和测试。

```bash
# 启动 Chrome（带远程调试端口，macOS）
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --headless --disable-gpu --window-size=1440,900

# 获取 tab 列表
curl http://localhost:9222/json
# 然后通过 WebSocket 发送 CDP 命令（Page.captureScreenshot 等）
```

优先用 bun 脚本直接调用 CDP WebSocket API，不依赖 Playwright。

---
> Source: [xuliuzhe/Nexora](https://github.com/xuliuzhe/Nexora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
