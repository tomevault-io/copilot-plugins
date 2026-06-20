## flowos

> 如流 (Flow OS) v5 — 基于 OpenClaw 的个人生活管理 Agent 团队。3 个 Agent（悦悦、军师、匠人）通过晨间/晚间/周度/月度节律，提供"确定感"陪伴。安装: bash install.sh


# 如流 (Flow OS)

> 你的个人生活管理 Agent 团队

## 简介

如流是一个运行在 OpenClaw 上的完整产品包，包含 3 个协作的 Agent：

- **悦悦**：你的知己，入口 Agent，给选项不给命令
- **军师**：数据分析师，周度/月度复盘，发现行为模式
- **匠人**：执行手艺人，文档/数据/代码/信息制作

## 核心功能

- **晨间主链路**：一次对话完成能量判定 → 思绪清空 → 自动排程
- **决策分析**：读取用户档案 + 原则库，分析利弊
- **Heartbeat 巡检**：每 15 分钟自动检测推迟和过度工作
- **晚间回顾**：每日任务总结推送
- **周度复盘**：四问报告（做了什么/做得好/没做好/怎么改）
- **月度战略**：方向校准 + 趋势分析 + 档案精炼

## 安装

```bash
bash install.sh                       # 交互式安装
bash install.sh --config config.env   # 非交互式（从配置文件读取）
bash install.sh --demo                # 演示模式（零配置）
```

10 步自动完成部署，支持可选插件（memory-lancedb-pro / ticktick-mcp）。

## 前置条件

- OpenClaw 已安装
- 飞书应用（App ID + App Secret，可选）
- Tavily API Key（可选，用于联网搜索）
- TickTick 无需配置，演示账号已预置

## 包含什么

- 3 个 Agent 的完整 SOUL.md + AGENTS.md（悦悦还包含 HEARTBEAT.md）
- 22 个 Skills 的完整 SKILL.md（悦悦 18、军师 5、匠人 4）
- 共享大脑模板（STATE/ + MEMORY/ + INSIGHTS/ + USER_MODEL/ + ANALYSIS/ + OUTPUTS/）
- 11 个 Cron Jobs（6 悦悦 + 5 军师）
- install.sh 一键部署脚本（支持 --config / --demo / --addon 参数）
- 独立安装脚本：install-memory-lancedb.sh / install-ticktick-mcp.sh
- validate-install.sh 安装后验证
- register-crons.sh Cron 批量注册

## 技术栈

- OpenClaw（Agent 编排）
- TickTick MCP（任务管理，预配置演示账号）
- Tavily MCP（联网搜索）
- Feishu MCP（用户通道）
- sessions_spawn + sessions_yield（匠人子 Agent 调度）
- Heartbeat（周期巡检）
- memory-lancedb-pro（可选，向量长期记忆）

---
> Source: [picsky/flowos](https://github.com/picsky/flowos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
