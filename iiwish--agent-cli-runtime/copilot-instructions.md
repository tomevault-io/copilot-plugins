## agent-cli-runtime

> - 本项目的主要交流、需求文档、设计文档、任务说明和 Codex 输出默认使用中文。

# AGENTS.md

## 语言约定

- 本项目的主要交流、需求文档、设计文档、任务说明和 Codex 输出默认使用中文。
- API 名、CLI 名、模型名、协议名、错误码、代码标识符、文件路径、命令、日志关键词等技术关键词可以保留英文。
- 面向用户的文档应优先保证中文可读性，不为了翻译而翻译稳定技术名词。

## Scope Calibration

- Match governance effort to task type. Do not turn broad, low-risk copy/i18n/content sweeps into dozens of micro governed tasks unless the user explicitly asks for that granularity.
- For i18n, wording, help content, labels, tooltips, and other broad text-only work, prefer an audit-and-batch workflow:
  1. classify visible strings by priority,
  2. batch related fixes by surface or user workflow,
  3. run representative tests plus key parity/type checks,
  4. summarize residual risks plainly.
- Use per-item task/evidence only for high-risk behavior, data, security, sync, auth, send/delete/archive, release, AI autonomy, or user-impacting logic changes.
- Keep the user's real goal in view. If the request is "add Chinese i18n," optimize for a usable Chinese product, not perfect documentation choreography.
- Before adding more process, ask: does this reduce product risk, or is it workflow theater?

## Git 工作流

- `0.1.0-alpha.1` 已发布后，默认不要直接在 `main` 上提交后续改动。
- 默认流程是从当前目标分出 `codex/<short-topic>` 分支，完成实现、验证和本地提交后，通过 PR 合并。
- release、workflow、schema、README/docs、package boundary、publish/runbook、evidence 相关改动必须优先走分支 + PR，因为这些内容会影响下一次 release evidence 或 npm package 内容。
- 只有用户明确要求“直接提交到 main”或“在当前分支提交”时，才可以跳过分支/PR 流程。
- 合并到 `main` 后，涉及 release/published verification 的改动需要重新触发 fresh workflow；历史 workflow run 只能证明自己的 head SHA。

## i18n Boundary

- Translate user-visible application chrome: navigation, settings, dialogs, errors, empty states, help content, command labels, tooltips, and status messages.
- Preserve technical identifiers and source content by default: protocol names, product/brand names, model names, API names, keyboard keys, URLs, search syntax, logs, email body text, email addresses, MIME/.eml headers, and server/library diagnostics unless explicitly user-facing and safely mappable.

---
> Source: [iiwish/agent-cli-runtime](https://github.com/iiwish/agent-cli-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
