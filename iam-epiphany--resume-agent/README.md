# AI instruction files for resume-agent

> Sourced from [iam-epiphany/resume-agent](https://github.com/iam-epiphany/resume-agent), graded against the public Tome Standard and kept consistent across every major platform by [TomeVault](https://tomevault.io)

面向"面试官 × 简历主人公"场景的个人简历 RAG 问答 Agent：把简历、证书、荣誉、项目介绍放进知识库，面试官可对简历任意提问（自我介绍、项目深挖、技术八股、HR 素质、简历细节），系统以第一人称自然作答。核心设计是"宽松推理 + 诚实标注"——硬事实（数字/日期/证书名）必须来自检索，证据不足时强制标注"根据现有知识库推测"而非编造。支持多轮追问记忆、检索兜底链、置信度分级回答、访问码闸门与 IP 限流防刷。

## GitHub Copilot Config

The `copilot-instructions.md` file in this directory is the project config converted for GitHub Copilot.
Original source: `AGENTS.md` in [iam-epiphany/resume-agent](https://github.com/iam-epiphany/resume-agent).

## Also available for

- **Claude Code** — `CLAUDE.md`
- **GitHub Copilot** — `copilot-instructions.md`
- **Cursor** — `project-config.mdc`
- **Gemini CLI** — `GEMINI.md`
- **Windsurf** — `project-config.md`

From [iam-epiphany/resume-agent](https://github.com/iam-epiphany/resume-agent) — a repo with 29+ stars on GitHub.

---

Install this config instantly:
```
npx tomevault install iam-epiphany/resume-agent
```
Source: [github.com/iam-epiphany/resume-agent](https://github.com/iam-epiphany/resume-agent).

<!-- genome:a-i-s -->
