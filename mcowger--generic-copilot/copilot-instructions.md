## generic-copilot

> For comprehensive instructions on working with this repository, please refer to [AGENTS.md](../AGENTS.md).

# GitHub Copilot Instructions

For comprehensive instructions on working with this repository, please refer to [AGENTS.md](../AGENTS.md).

The AGENTS.md file contains:
- Project overview and architecture
- Tech stack and dependencies
- Development workflow and build commands
- Coding conventions and critical rules
- Guidelines for adding new features
- Security best practices

**Quick Reference - Critical Rules:**
- 🛑 **NEVER** use `console.log` - ALWAYS use `logger` from `src/outputLogger.ts`
- Use `vscode.secrets` for API keys and sensitive data
- This extension uses VS Code Proposed APIs
- The project has two parts: root extension and `webview-ui/` - both need `npm install`

---
> Source: [mcowger/generic-copilot](https://github.com/mcowger/generic-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
