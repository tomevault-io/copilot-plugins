## openclaw-dashboard

> OpenClaw Dashboard 是 OpenClaw AI 网关框架的实时监控面板。它通过 WebSocket 连接到 OpenClaw Gateway，采集会话、Token 用量、费用等指标，并以赛博朋克风格的 Web 界面实时展示。

# AGENTS.md

## 项目概述

OpenClaw Dashboard 是 OpenClaw AI 网关框架的实时监控面板。它通过 WebSocket 连接到 OpenClaw Gateway，采集会话、Token 用量、费用等指标，并以赛博朋克风格的 Web 界面实时展示。

界面语言为中文。

## 仓库结构

这是一个 npm workspaces 单仓库，包含两个包：

- `packages/server/` — Node.js 后端（Express + WebSocket），负责连接网关、解析会话日志、聚合指标
- `packages/web/` — React 前端（Vite 构建），负责实时数据展示

构建产物统一输出到 `dist/`，由 server 托管前端静态资源。

## 开发与构建

参考根目录 `README.md` 获取环境变量、构建命令和部署方式。开发时需同时启动 server 和 web 两个 dev 进程，Vite 会将 API 和 WebSocket 请求代理到后端。

## 后端架构导引

后端源码在 `packages/server/src/`，入口是 `server.ts`。

关键模块（按数据流顺序）：

1. **网关连接** — 查看 `gateway-client.ts`（WebSocket RPC）和 `identity.ts`（设备认证）
2. **会话日志解析** — 查看 `activity-tracker.ts`（文件监听与活动提取）和 `session-parser.ts`（JSONL 解析与噪音过滤）
3. **任务摘要生成** — 查看 `task-summarizer.ts`（调用网关 AI 能力生成中文任务标题）
4. **指标聚合与推送** — 查看 `metrics.ts`（数据聚合）和 `server.ts`（定时广播）

配置项集中在 `config.ts`，通过环境变量控制。

## 前端架构导引

前端源码在 `packages/web/src/`，入口是 `main.tsx`。

- **数据获取** — `hooks/useMetrics.ts` 通过 WebSocket 接收后端推送的指标，是所有组件的数据源
- **页面布局** — `App.tsx` 定义卡片网格布局
- **各卡片组件** — 在 `components/` 目录下，文件名与功能直接对应（TokenUsage、Today、CostBreakdown、Sessions、TaskLog、Activity 等）
- **类型定义** — `lib/types.ts` 定义了前后端共享的数据结构
- **工具函数** — `lib/format.ts` 包含数字格式化、时间处理等
- **样式** — `style.css` 是全局样式，采用赛博朋克暗色主题

## 数据流

```
OpenClaw Gateway ←WebSocket RPC→ Server (采集+解析) ←WebSocket 推送→ Frontend (渲染)
                                    ↑
                          会话日志文件 (JSONL)
                       ~/.openclaw/agents/main/sessions/
```

服务端每 10 秒聚合一次指标并广播给所有前端连接。

## 注意事项

- 会话日志的 JSONL 格式和字段结构可能随 OpenClaw 版本变化，相关解析逻辑集中在 `session-parser.ts`
- 网关认证协议（当前为 v3，ed25519 签名）的细节见 `gateway-client.ts` 和 `identity.ts`
- 前端无路由，是单页单屏的仪表盘应用

## Git 提交规范

提交消息使用 Conventional Commits 格式，中文描述：

```
<type>(<scope>): <简要描述>
```

**type** 常用值：`feat`、`fix`、`refactor`、`style`、`chore`、`docs`

**scope** 对应功能模块，如 `activity`、`task-log`、`ui`、`header`、`footer` 等。影响范围明确时必须标注 scope。

描述应简明扼要，说清"做了什么"即可。如涉及英文术语（如组件名、技术概念）可直接使用英文，无需强行翻译。

### AI 辅助开发时的提交要求

- 按合理粒度自行提交——一个逻辑完整的变更对应一个 commit，不要把不相关的改动混在一起，也不要把一个功能拆得过碎
- 每个 commit message 末尾必须附上 Co-Authored-By trailer，标明模型名称和版本号，例如：

```
feat(activity): 增加工具调用耗时展示

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

```
fix(task-log): 修正空会话导致的崩溃

Co-Authored-By: GPT-5.4 <noreply@openai.com>
```

---
> Source: [xingrz/openclaw-dashboard](https://github.com/xingrz/openclaw-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
