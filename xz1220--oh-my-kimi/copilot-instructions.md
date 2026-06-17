## oh-my-kimi

> oh-my-kimi 的目录入口，包含面向 Kimi CLI 的 agent、skill、hook 与 MCP 套件，衍生自 oh-my-* 谱系。


# oh-my-kimi

当你需要在 Kimi CLI 内理解或启用 oh-my-kimi 目录时，使用本 skill。

## 已安装的内容

- Agent 位于 `agents/`，可通过 `kimi --agent-file <repo>/agents/oh-my-kimi.yaml` 或 `kimi-omk` 包装器加载。
- Skill 位于 `skills/<name>/SKILL.md`，运行 `scripts/install.sh` 后可作为 `/skill:<name>` 使用。
- Hook 模板位于 `hooks/`，由完整安装器追加到 `~/.kimi/config.toml`。
- MCP 辅助脚本位于 `mcp/`，需要相应集成时可单独运行。

## 推荐工作流

1. 需求模糊时使用 `/skill:deep-interview`。
2. 实现前需要一份已落地的计划时使用 `/skill:ralplan`。
3. 需要持续执行直至验证通过时使用 `/skill:ralph`。
4. 并行的专家协作有价值时使用 `/skill:team`。
5. 在最终质量关卡使用 `/skill:verify` 或 `/skill:visual-verdict`。

## Agent 路由

小改动优先直接动手。仅当任务能从隔离的上下文或并行调查中获益时，再使用 Agent subagent：

- `explore` 用于只读代码搜索。
- `planner` 或 `architect` 用于设计方案。
- `executor` 用于实现。
- `critic`、`verifier`、`code-reviewer`、`security-reviewer`、`qa-tester` 或 `test-engineer` 用于评审与验证。

---
> Source: [xz1220/oh-my-kimi](https://github.com/xz1220/oh-my-kimi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
