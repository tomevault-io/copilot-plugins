## chatgpt-md

> This folder contains repository automation, primarily GitHub Actions workflows.

# GitHub Configuration

This folder contains repository automation, primarily GitHub Actions workflows.

## Files

- `workflows/ci.yml` - CI validation for the plugin

## Rules for agents

- Do not weaken CI to make failures pass.
- Keep CI aligned with documented local validation commands in `AGENTS.md`, root `CLAUDE.md`, and `docs/development.md`.
- Prefer npm commands:

```bash
npm run build
npm test -- --runInBand
npm run lint
```

- If changing Node versions, dependency installation, caching, or package-manager behavior, verify `package.json` and lockfile expectations.
- Do not add secrets directly to workflow files. Use GitHub Actions secrets and document required names.
- Avoid running costly provider/API tests in CI unless explicitly requested; model/tool whitelist scripts are manual maintenance workflows.

## Validation

For workflow edits, inspect syntax and run equivalent local commands where practical. Mention if GitHub-hosted behavior cannot be fully validated locally.

---
> Source: [bramses/chatgpt-md](https://github.com/bramses/chatgpt-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
