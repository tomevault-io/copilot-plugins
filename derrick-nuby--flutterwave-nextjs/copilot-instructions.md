## flutterwave-nextjs

> A modern, type-safe Next.js application template for integrating **Flutterwave** payment gateway. Built with React, TypeScript, and Tailwind CSS, this project provides a solid foundation for accepting payments across Africa, including mobile money solutions.

# Flutterwave Payment Integration with Next.js

A modern, type-safe Next.js application template for integrating **Flutterwave** payment gateway. Built with React, TypeScript, and Tailwind CSS, this project provides a solid foundation for accepting payments across Africa, including mobile money solutions.
hen working with code in this repository.

## CRITICAL: Before ANY Task Submission

**ALWAYS run this command before committing or creating PRs:**

If it fails, fix the issues first!

```bash
pnpm format && pnpm lint && pnpm type-check
```

## Package Manager

**Use `pnpm` exclusively** - not npm or yarn.

## Tech Stack

- Next.js 16 (App Router) + React 19 + TypeScript (strict mode)
- Tailwind CSS v4 + shadcn/ui (New York style)
- Biome (replaces ESLint + Prettier)
- Flutterwave payment gateway integration

## Essential Commands

```bash
pnpm dev              # Development server
pnpm build            # Production build
pnpm format           # Format with Biome
pnpm lint             # Lint and auto-fix with Biome
pnpm type-check       # TypeScript validation
```

## Branch Naming Rules

**Use these prefixes:**

- `feature/` - New features
- `bugfix/` - Bug fixes
- `release/` - Release branches

**DO NOT use `claude/` prefix.**

**Rules:**

- Short and descriptive (e.g., `feature/payment-integration`)
- Use hyphens (`-`) for word separation
- No numbers-only or generic names (avoid `feature/123` or `feature/fix`)

## Path Aliases

- `@/*` maps to `./src/*`
- Use `@/components`, `@/lib/utils`, `@/hooks` as configured

## Important Notes

- **Biome skips linting `src/components/ui`** (auto-generated shadcn components)
- UI components use `cn()` utility from `@/lib/utils` for class merging
- TypeScript strict mode enabled - no type errors allowed

## Resources

### Flutterwave

- [Official Documentation](https://developer.flutterwave.com/docs)
- [React Integration Guide](https://developer.flutterwave.com/docs/flutterwave-react-v3)
- [API Reference](https://developer.flutterwave.com/reference)
- [Dashboard](https://dashboard.flutterwave.com)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/derrick-nuby)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/derrick-nuby)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
