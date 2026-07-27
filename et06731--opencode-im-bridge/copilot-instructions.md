## opencode-im-bridge

> - **默认模式**: 机器人表现为 `opencode` 的影子，所有输入被视为在 TUI 中键入。

# 机器人行为规范 (Bot Behavior)

## 1. 响应逻辑 (Response Logic)

- **默认模式**: 机器人表现为 `opencode` 的影子，所有输入被视为在 TUI 中键入。
- **自动绑定**: 机器人会尽力“接续”最近的 TUI 活动，减少用户的配置负担。
- **上下文感知**: 每个聊天 Thread 拥有独立的记忆，互不干扰（除非显式指定绑定到同一 Session）。

## 2. 交互体验原则

- **流式优先**: 始终尝试展示即时的文字流。
- **可见的工具调用**: 当 Agent 使用工具时，会有明确的 UI。
- **简洁性**: 飞书消息中默认渲染 Markdown，确保代码块和列表排版自然。

## 3. 设计约束

- **文件大小**: 受平台限制，附件下载最大支持 50MB。
- **超时处理**: 默认 SSE 连接超时为 5 分钟，超时后会自动进入空闲状态或上报错误。

---
> Source: [ET06731/opencode-im-bridge](https://github.com/ET06731/opencode-im-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
