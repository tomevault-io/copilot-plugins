## project-general

> Repository-wide Cursor guidance for Allure TestOps MCP


# Allure TestOps MCP - General Rules

- Use TypeScript ESM patterns (`import`/`export`) and keep changes aligned with existing code style.
- Preserve public tool contracts; do not rename tool names or change input/output shapes unless explicitly requested.
- Prefer small, focused edits in `src/tools/` and shared helpers over duplicate logic.
- Keep auth behavior stable: this project exchanges API token to JWT and reuses cached JWT until near expiry.
- Avoid introducing new dependencies unless required; prefer existing utilities and standard library first.

## Validate Before Finishing

- Run `npm run build` after meaningful TypeScript changes.
- Run `npm run lint` and fix issues in touched files.
- Run `npm run test` (or targeted integration tests) for behavior changes.

## Safety

- Never commit secrets (`.env`, real tokens, URLs with credentials).
- Keep example docs generic and safe for public repository use.

---
> Source: [armanayvazyan/allure-testops-mcp](https://github.com/armanayvazyan/allure-testops-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
