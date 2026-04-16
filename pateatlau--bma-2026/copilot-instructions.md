## bma-2026

> Code quality standards and restrictions


# Code Quality Standards

## General Principles

- This is a real production-style Expo project
- Code must be runnable, not illustrative
- Prefer correctness and clarity over cleverness
- Think through architecture before writing code
- Fix issues fully, not partially

## Restrictions

- Do NOT use web-only libraries (no react-router-dom, no Next.js)
- Do NOT use deprecated APIs
- Do NOT invent libraries
- Do NOT trust client-side payment callbacks (use webhook only)
- Do NOT skip RLS policies on any table
- Do NOT use "any" type unless absolutely necessary with documented justification

## Quality Checklist

- Maintain clean file structure
- Avoid duplicated logic (DRY principle)
- Ensure navigation and providers are wired correctly
- Handle platform differences intentionally
- Run `npm run check` before committing

## Commands

```bash
npm run check        # typecheck + lint + format:check
npm run lint:fix     # Auto-fix linting issues
npm run format       # Format code with Prettier
npm run test         # Run unit tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pateatlau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
