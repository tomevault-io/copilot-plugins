## code-quality

> Lint, format, and type-check after medium/large changes


# Code Quality Checks

After every medium or large change, verify zero errors from all three:

1. **Lint**: `npm run lint` — ESLint must report no errors.
2. **Format**: `npm run format:check` — Prettier must report no unformatted files. Run `npm run format` to fix.
3. **Types**: `npm run build` — TypeScript must compile without errors.

These checks are mandatory before considering a task complete.
The IDE auto-formats on save (Prettier) and auto-fixes lint issues (ESLint),
but always run the CLI checks to confirm.

---
> Source: [collectioneur/readme-aura](https://github.com/collectioneur/readme-aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
