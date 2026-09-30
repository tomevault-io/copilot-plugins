# AI instruction files for Grader-agent

> Sourced from [erictrainnlpinxihu/Grader-agent](https://github.com/erictrainnlpinxihu/Grader-agent), graded against the public Tome Standard and kept consistent across every major platform by [TomeVault](https://tomevault.io)

Grader 的 agent 采用五阶段主循环：perceive→plan→act→observe→respond，由 harness 驱动。模型只决定意图与置信度，路由、工具集、风险等级由代码钉死。act 分只读 ReAct、RAG、人工提案、批量初批。四个写动作不在工具表，只能产提案。批改输出结构化 GradingDraft，逐条绑定 rubric 与段落 hash，保证可重放。

## GitHub Copilot Config

The `copilot-instructions.md` file in this directory is the project config converted for GitHub Copilot.
Original source: `CLAUDE.md` in [erictrainnlpinxihu/Grader-agent](https://github.com/erictrainnlpinxihu/Grader-agent).

## Also available for

- **Codex** — `AGENTS.md`
- **GitHub Copilot** — `copilot-instructions.md`
- **Cursor** — `project-config.mdc`
- **Gemini CLI** — `GEMINI.md`
- **Windsurf** — `project-config.md`

From [erictrainnlpinxihu/Grader-agent](https://github.com/erictrainnlpinxihu/Grader-agent) — a repo with 9+ stars on GitHub.

---

Own this repo? Install the TomeVault Relay to keep every platform's copy in sync on every push: [https://tomevault.io/install](https://tomevault.io/install).

<!-- genome:a-c-s -->
