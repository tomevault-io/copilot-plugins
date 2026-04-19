## create-helix-app

> `create-helix` is a TUI-powered CLI scaffolder for HELiX enterprise web component projects. It supports 8 framework targets and 4 Drupal presets.

# CLAUDE.md -- create-helix

## Project Overview

`create-helix` is a TUI-powered CLI scaffolder for HELiX enterprise web component projects. It supports 8 framework targets and 4 Drupal presets.

## Tech Stack

- TypeScript (strict mode)
- Node.js >= 20 (ESM)
- @clack/prompts (TUI)
- Vitest (testing)
- ESLint 9 (flat config) + Prettier

## Quick Reference

```bash
npm run verify     # lint + format:check + type-check (MANDATORY before push)
npm run test       # Run all tests
npm run build      # Compile TypeScript
npm run preflight  # verify + test (full pre-push check)
```

## Non-Negotiables

1. **TypeScript strict** -- No `any`. No `@ts-ignore`.
2. **`npm run verify` before every push** -- Zero CI cycles wasted.
3. **No secrets in code** -- This is a PUBLIC repo. Zero tolerance.
4. **Tests for all scaffolding logic** -- Every framework, every preset.

## Git Rules

- `npm run verify` must pass before push (enforced by Husky pre-push hook)
- Never use `--no-verify`
- Commit messages: concise, imperative ("Add Vue support", not "Added")
- Branch strategy: `feature/* -> dev -> staging -> main`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bookedsolidtech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
