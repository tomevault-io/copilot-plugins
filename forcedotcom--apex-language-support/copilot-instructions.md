## doc-maintenance

> Delegate to doc-maintenance subagent when code/config/scripts change; catches code→doc drift



When code is edited (or code changes are in conversation context), delegate to the [doc-maintenance agent](../../agents/doc-maintenance.md) with `run_in_background: true`. Subagent fixes docs directly; no report-back required.

## Activate on changes to

- `**/*.ts`, `**/*.tsx`
- `**/package.json`
- `**/esbuild.config.*`
- `scripts/**`
- `**/.vscodeignore`
- `**/.vscode/**`
- `**/tsconfig*.json`
- `.esbuild-web-extra-settings.json`
- `.github/**`

## Scope

- In scope: `.claude/skills/`, `.claude/agents/`, `.cursor/rules/`, `docs/`, `contributing/`, `packages/**/README.md`
- Excluded: `**/*.plan.md`, `**/plan.md`

---
> Source: [forcedotcom/apex-language-support](https://github.com/forcedotcom/apex-language-support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
