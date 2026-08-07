## kflow

> kflow 是一个跨平台 AI 编码工作流技能集合，基于 AgentSkills.io 标准。技能在 Claude Code/Cursor/Codex/OpenCode 上均可运行。本文件为 AI 编码 agent 提供工作指引。

# AGENTS

kflow 是一个跨平台 AI 编码工作流技能集合，基于 AgentSkills.io 标准。技能在 Claude Code/Cursor/Codex/OpenCode 上均可运行。本文件为 AI 编码 agent 提供工作指引。

## 项目性质

- 本项目是一个**技能包仓库**，不是应用项目
- 包含 25 个 `k-*` 技能（AgentSkills.io 标准格式），一个 `browser-bridge` 独立技能
- 技能由 `k-onboard` 复制到目标项目的 `.kflow/skills/` 下，各平台 AI 通过 `AGENTS.md` 中的路由表按需读取
- 本仓库是技能的**开发和维护空间**，不是技能的执行环境

## 工作方式

- 所有 `k-*` 目录是 AgentSkills.io 标准格式（SKILL.md + 周边文件）
- 修改技能行为 → 改对应 `k-*/SKILL.md`
- 修改共享参考 → 改 `k-onboard/reference/` 模板

## 约束

- 技能间不耦合，每个 skill 独立可用
- 单 md 不超过 300 行
- 跨技能共享文档走 `k-onboard/reference/` → `.kflow/reference/`
- 修改技能后检查是否有其他技能需要同步更新表述

---
> Source: [kunbo928/kflow](https://github.com/kunbo928/kflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
