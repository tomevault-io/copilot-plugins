## api-worker

> 本文件定义本仓库内 Agent 的统一执行规范。

# AGENTS.md

本文件定义本仓库内 Agent 的统一执行规范。

## 1. 基本原则

- 变更优先最小化，避免无关改动。
- 文档与代码保持一致；行为变化必须同步文档。
- 所有回答和提交说明使用中文（保留代码与 API 原文）。

## 2. 本地环境变量

Agent 本地运行前，若不存在 `.env.agent`，先参考 `.env.agent.example` 创建本地 `.env.agent`（模板仅作参考）。

## 3. 完成后必须执行

每次任务完成后，必须执行以下检查（按顺序）：

1. `bunx --bun biome format --write <changed-files>`
2. `bun run typecheck`
3. `bun run test`

如果任一步失败，先修复再交付。

## 4. 前后端验证规则

- 前端改动：必须使用自动化工具做最小可用回归（页面加载、关键交互、无控制台报错）。
- 后端改动：必须通过直接请求验证接口行为（`curl`/`Invoke-RestMethod`）。

## 5. 结果汇报要求

- 明确列出改动文件。
- 明确列出执行过的命令和结果（通过/失败）。
- 若有未完成项或风险，必须显式说明。
- 任务完成时，主动建议用户执行 `git commit`（若本次未代为提交）。

---
> Source: [lsy2246/api-worker](https://github.com/lsy2246/api-worker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
