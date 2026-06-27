## ai-skills

> 本仓库是个人 AI skill 源仓库。每个顶层 skill 目录必须可被 Codex、Claude Code、Cursor 复用。

# Codex 项目指令

本仓库是个人 AI skill 源仓库。每个顶层 skill 目录必须可被 Codex、Claude Code、Cursor 复用。

## Skill 规范

- `SKILL.md` 使用简体中文编写；必要关键词、命令、字段名可用英文。
- `SKILL.md` 保持短小，只写触发、核心流程、安全边界和必要命令；详细说明放 `README.md` 或 `references/`。
- 每个 skill 必须有 `README.md`，且包含 `## USAGE` 模块。
- `README.md` 面向人读；`SKILL.md` 面向模型运行，优先节省 token。
- 更新 `SKILL.md`、`agents/openai.yaml`、`entrypoints/`、`references/`、`scripts/` 后，必须同步到本地用户级 skills 目录。

## 三端识别

每个 skill 至少维护这些入口：

- Codex：`<skill>/SKILL.md`，同步到 `~/.codex/skills/<skill>/SKILL.md`。
- Claude Code：同一份 `<skill>/SKILL.md`，同步到 `~/.claude/skills/<skill>/SKILL.md`。
- Cursor：`<skill>/entrypoints/cursor/<skill>.mdc` 是源码入口；项目自动识别入口是 `.cursor/rules/<skill>.mdc`。
- Codex UI 元数据：`<skill>/agents/openai.yaml`。

`SKILL.md` 的 YAML frontmatter 只放必要字段：`name`、`description`。`description` 必须写清楚触发场景。

## 同步规则

当任意 skill 内容更新后，执行：

```bash
mkdir -p ~/.codex/skills ~/.claude/skills
cp -R <skill> ~/.codex/skills/
cp -R <skill> ~/.claude/skills/
cp <skill>/entrypoints/cursor/<skill>.mdc .cursor/rules/<skill>.mdc
```

同步前后用 `git status --short` 和目标目录快速核对。不要删除用户级目录中与当前任务无关的 skill。

## 变更检查

- 新增或更新 skill 后，确认 `SKILL.md`、`README.md`、`agents/openai.yaml`、Cursor `.mdc` 入口一致。
- 若 skill 只有旧的 `USAGE.md`，补成 `README.md` 并保留 `## USAGE`。
- 保持语言简洁，不写营销文案，不重复 `SKILL.md` 已有流程。

---
> Source: [yangin/ai-skills](https://github.com/yangin/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
