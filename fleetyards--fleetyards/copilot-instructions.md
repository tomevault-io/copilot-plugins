## fleetyards

> - **ESLint**: JavaScript/TypeScript linting


# Frontend Formatting

## Tools

- **ESLint**: JavaScript/TypeScript linting
- **Prettier**: Code formatting
- **Stylelint**: CSS/SCSS linting

## Commands

```bash
pnpm lint               # Run all linting (ESLint + Stylelint + TS + formatting)
pnpm lint:fix           # Fix auto-fixable issues
pnpm lint:js            # Fix JavaScript/TypeScript ESLint issues only
pnpm lint:ts            # TypeScript type checking for Vue/TypeScript files
pnpm format             # Format frontend with prettier
pnpm format:fix         # Format and fix frontend with prettier
```

## Important Notes

- **Use `pnpm lint:fix` for comprehensive fixing** of ESLint, CSS,
  and formatting issues
- **Use `pnpm lint:ts` for TypeScript type checking** for Vue and
  TypeScript files
- The `type-check` and `typecheck` command does not exist in this project

## Auto-formatting

- VSCode/Cursor auto-formats on save
- Pre-commit hooks enforce formatting
- Use `pnpm lint:fix` for bulk fixes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fleetyards)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fleetyards)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
