## multi-agent-coordinator

> 多 Agent 协调系统 - OpenClaw 原生 Skill

# Multi-Agent Coordinator Skill

多 Agent 协调系统 - OpenClaw 原生 Skill

## 工具依赖

- `sessions_send`: 用于向当前会话发送 spawn 指令
- `sessions_spawn`: 由 OpenClaw 自动解析并调用（通过自然语言指令）

## 功能

- ✅ 创建和管理多个子 Agent 协同完成任务
- ✅ 支持四种协作模式：流水线/并行/讨论/混合
- ✅ 支持混合工作流 (DAG 调度)
- ✅ 与 Web UI 配合提供可视化配置和监控
- ✅ 真实 sessions_spawn 集成（2026-04-08 完成）

## 使用方式

### 通过 Web UI
启动本地服务后访问 `http://localhost:3456`

### 通过自然语言
```
启动多 Agent 会话：
- 3 个 Agent：coder, reviewer, tester
- 模式：流水线
- 任务：重构用户认证模块
```

### 通过 Skill API
```javascript
import { coordinator } from './multi-agent-coordinator/src/index.js'

await coordinator.init()
await coordinator.startSession({
  sessionId: 'my-session',
  mode: 'parallel',
  agents: [
    { type: 'coder', role: '前端开发' },
    { type: 'reviewer', role: '代码审查' }
  ],
  task: '开发一个待办事项应用'
})
```

## 配置

详见 `config.json` 和 Web UI 界面

## 文件结构

```
multi-agent-coordinator/
├── SKILL.md              # 本文件
├── config.json           # 配置文件
├── src/
│   ├── index.js          # Skill 入口
│   ├── agentFactory.js   # Agent 工厂
│   ├── orchestrator.js   # 协调器
│   ├── communication.js  # 通信总线
│   └── aggregator.js     # 结果聚合
├── server/
│   ├── index.js          # HTTP 服务器
│   └── websocket.js      # WebSocket 服务
└── web-ui/               # React 前端 (阶段 2 创建)
```

---
> Source: [yijianw/Multi-Agent-Coordinator](https://github.com/yijianw/Multi-Agent-Coordinator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
