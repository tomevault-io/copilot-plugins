## web-ai

> This project uses webGenerate-generated knowledge docs.

## webGenerate

This project uses webGenerate-generated knowledge docs.

Rules:
- Before answering frontend business questions, read webAIDocs/routes.md if it exists
- For page feature refactor/development, first read webAIDocs/routes.md and the matched webAIDocs/page-xxx.md, then locate component/api/store files for code changes
- If routes/page docs are missing or outdated:
  - Codex uses `$webGenerate .`
  - Claude/OpenCode/Trae/Trae CN/Copilot/Gemini/Cursor/Antigravity use `/webGenerate .`
- After code changes, run incremental sync with the same trigger form and `--update`

---
> Source: [hyb654197815/web-ai](https://github.com/hyb654197815/web-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
