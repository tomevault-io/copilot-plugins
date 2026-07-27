## salesforcedx-vscode

> Delegate to doc-maintenance subagent when code/config/scripts change; catches code→doc drift


When agent edits code (or has code changes in context), delegate to doc-maintenance subagent (@.claude/agents/doc-maintenance.md) with `run_in_background: true`. Subagent fixes docs directly; no need to report back for parent to fix.

---
> Source: [forcedotcom/salesforcedx-vscode](https://github.com/forcedotcom/salesforcedx-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
