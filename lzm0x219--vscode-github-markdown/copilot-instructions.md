## vscode-github-markdown

> - Keep changes minimal and directly tied to GitHub Markdown parity.

# Conventions

- Keep changes minimal and directly tied to GitHub Markdown parity.
- Use built-in markdown.markdownItPlugins, markdown.previewScripts, and markdown.previewStyles hooks when sufficient.
- Do not import Node-only APIs into src runtime unless web-extension support is intentionally changed; Node APIs are acceptable in scripts/.
- README.md is English default; keep README.zh-CN.md aligned for shared content.
- CHANGELOG entries must satisfy docs/CHANGELOG_STYLE_GUIDE.md admission rules and describe only user-observable/actionable outcomes.
- Do not raise engines.vscode unless implementation requires newer APIs.
- Tests should exercise observable rendered HTML or extension lifecycle through the module interface; full-chain tests are needed for renderer-rule ordering interactions.
- GitHub Issues are the issue/PRD source; follow docs/agents/issue-tracker.md.

---
> Source: [lzm0x219/vscode-github-markdown](https://github.com/lzm0x219/vscode-github-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
