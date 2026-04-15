## zombie-digital

> description: Package manager rules for Zombie.Digital project - always use bun instead of npm/yarn/pnpm

---
description: Package manager rules for Zombie.Digital project - always use bun instead of npm/yarn/pnpm
alwaysApply: true
---

# Package Manager Rules

Always use bun for package management in this project.

## Commands to use:
- `bun add package-name` for package installation
- `bunx package-name` for package execution  
- `bun run script-name` for running scripts

## Commands to avoid:
- ❌ `npm install`
- ❌ `npx package-name`
- ❌ `yarn add`
- ❌ `pnpm add`

## Examples:
```bash
# ✅ Correct
bun add @types/node
bunx create-next-app
bun run dev

# ❌ Incorrect
npm install @types/node
npx create-next-app
npm run dev
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joenilan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joenilan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
