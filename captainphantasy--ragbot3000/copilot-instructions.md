## ragbot3000

> **DO NOT** modify these files without explicit user approval:

# RAGBOT3000 Production Rules

## CRITICAL: Dependency Lock

**DO NOT** modify these files without explicit user approval:
- `package.json`
- `package-lock.json`
- Any `*.lock` files

**DO NOT** run these commands:
- `npm install <package>`
- `npm add`
- `npm remove`
- `yarn add`
- `pnpm add`

**DO NOT** add new linter/formatter tools:
- No biome, oxlint, secretlint, eslint configs
- No prettier configs
- No new devDependencies

## This is a PRODUCTION environment

- Every git commit deploys to Railway
- Dependencies are frozen
- Only modify application code, persona files, and knowledge base

## Allowed modifications

✅ `/persona/` - Prompts and behavior
✅ `/knowledge/` - RAG documents and index
✅ `/components/` - UI components
✅ `/services/` - Application services (no new imports)
✅ `App.tsx` - Main app
✅ `README.md` - Documentation

## Before ANY git operation

1. Confirm with user
2. Use conventional commit format
3. Never force push

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CaptainPhantasy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
