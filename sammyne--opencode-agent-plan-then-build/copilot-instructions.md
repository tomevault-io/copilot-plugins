## opencode-agent-plan-then-build

> 本仓库是 OpenCode「先规划后编码」工作流的 agent 定义文件集合。没有应用代码，没有构建/测试/lint 命令。产出物是 `.opencode/agents/` 下的两个 agent 配置，设计为拷贝到其他项目中使用。

# AGENTS.md

## 仓库说明

本仓库是 OpenCode「先规划后编码」工作流的 agent 定义文件集合。没有应用代码，没有构建/测试/lint 命令。产出物是 `.opencode/agents/` 下的两个 agent 配置，设计为拷贝到其他项目中使用。

## 架构

```
.opencode/agents/
  planner.md   — 需求分析 + TODO 生成（仅可写入 .opencode/plans/**）
  builder.md   — 按 TODO 列表执行编码（可编辑 plans 中的 todo.md 及所有项目文件）
```

**工作流**：用户激活 planner → 生成 `requirements.md` + `todo.md` 到 `.opencode/plans/{名称}/` → 用户确认 → 切换到 builder agent → 按依赖顺序执行任务。

## 关键约束

- **planner** 无 bash 权限；仅可写入 `.opencode/plans/**`
- **builder** 仅可编辑 plans 中的 `todo.md`（标记任务完成），不可修改其他规划文件；对项目文件有完整的 bash/write/edit 权限
- 两个 agent 均从**目标项目**（即 agent 文件被拷贝到的项目）的 `AGENTS.md` 读取项目结构信息，而非本仓库的 `AGENTS.md`
- agent 文件头部的 YAML frontmatter（`---` 之间）控制权限和工具访问，格式损坏会导致 agent 失效

## 编辑须知

- 修改 agent 文件时，必须保持 YAML frontmatter 结构不变（`description`、`mode`、`temperature`、`tools`、`permission`）
- `opencode.json` 已被 gitignore，属于用户本地配置，不要提交
- `.opencode/package.json` 固定了 `@opencode-ai/plugin` 版本；`.opencode/` 内的 `node_modules` 和 lockfile 也已被 gitignore

---
> Source: [sammyne/opencode-agent-plan-then-build](https://github.com/sammyne/opencode-agent-plan-then-build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
