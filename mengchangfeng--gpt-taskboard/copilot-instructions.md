## gpt-taskboard

> 本仓库提供两篇按顺序使用的教程：先用 AgentDock 把网页版 ChatGPT 安全连接到电脑，再用项目文档、Task Board、Git worktree 和定时唤醒持续开发。

# GPT-TaskBoard — Agent Rules

本仓库提供两篇按顺序使用的教程：先用 AgentDock 把网页版 ChatGPT 安全连接到电脑，再用项目文档、Task Board、Git worktree 和定时唤醒持续开发。

## 阅读顺序

首次接手先阅读：

1. `README.md` — 新版教程入口与验收顺序。
2. `tutorials/01-agentdock-mcp-and-web-chatgpt.md` — AgentDock 官方安装主线与自建公网入口。
3. `tutorials/02-task-board-and-scheduled-development.md` — Task Board、worktree、验收和定时开发协议。
4. `examples/note-app-walkthrough.md` — 从 READY 到 DONE 的虚构案例。

修改模板或 Prompt 时，再阅读 `templates/project/` 与 `templates/prompts/` 中的对应文件。

## 维护规则

- Task Board 核心协议保持工具无关；AgentDock 是教程采用的电脑 MCP 运行时，不是项目状态来源。
- `tutorials/01-agentdock-mcp-and-web-chatgpt.md` 与 `config-examples/` 必须以官方 AgentDock 文档为权威依据，并保持脱敏；不得写入真实域名、IP、个人路径、Token、密码、私钥或配置原文。
- 本仓库提供参考机制和 Prompt，不提供安装器，也不要求目标项目机械复制固定文件结构。
- `templates/prompts/adopt-workflow.md` 必须要求 Agent 优先阅读并遵循目标项目已有文档约定，只补真正缺失的职责。
- `templates/prompts/scheduled-developer.md` 只依赖项目中可以被明确找到的状态，不依赖聊天记忆。
- 重要行为变化同步更新 `README.md`、对应教程、模板或示例。
- 每次任务完成都应提交 Git；存在远端时再推送，未配置远端时不要假装已推送。

---
> Source: [mengchangfeng/GPT-TaskBoard](https://github.com/mengchangfeng/GPT-TaskBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
