## codepulse

> - Language: TypeScript ESM (strict mode, Node 22+)

# Project — Cursor IDE Rules
# For detailed rules, see .clinerules/ directory

## Quick Reference
- Language: TypeScript ESM (strict mode, Node 22+)
- Package Manager: pnpm 10+
- Build: `pnpm build` (tsc)
- Lint: `pnpm lint` (oxlint)
- Test: `pnpm test` (vitest, V8 coverage 70%)
- QA: `pnpm qa` (full pipeline)
- Fix: `pnpm fix` (auto-fix all)

## Naming
- Files: kebab-case | Classes: PascalCase | Functions: camelCase

## Security (Non-Negotiable)
- NO hardcoded secrets/API keys/tokens
- NO `eval()` or `Function()` constructor
- NO `console.log` for debugging
- NO `any` without justification comment
- NO mock/placeholder data in src/ files (tests only)
- Validate all external input (Zod at boundaries)

## Full Rules
See `.clinerules/` for comprehensive standards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/velonone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
