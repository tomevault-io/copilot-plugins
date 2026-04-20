## random-number-generator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server on port 3000
npm run build    # Build for production
npm run test     # Run tests with Vitest
npm run deploy   # Build and deploy to Cloudflare Workers
```

## Architecture

This is a TanStack Start application with Cloudflare Workers deployment. It's a random number generator that fetches entropy seeds from external sources (quantum/atmospheric noise APIs) and falls back to browser CSPRNG.

### Key Directories

- `src/routes/` - File-based routing (TanStack Router). `__root.tsx` is the root layout, `index.tsx` is the home page
- `src/components/ui/` - Radix-based UI primitives (shadcn/ui style)
- `src/components/features/` - Feature components (e.g., `RandomGenerator.tsx`)
- `src/components/layout/` - Layout components (`RootLayout.tsx`, `AppSidebar.tsx`)
- `src/services/` - Business logic and API clients

### Random Number Generation Flow

The `RandomGenerator` component uses `generateRandomNumbers()` from `src/services/randomService.ts` which fetches random numbers directly with priority fallback:
1. **Priority 1**: ANU Quantum RNG (`qrng.anu.edu.au`)
2. **Priority 2**: Random.org Atmospheric Noise
3. **Priority 3**: Web Crypto API (CSPRNG) - local fallback

Each generation attempt tries sources in order until one succeeds. The source used is displayed in the result UI.

### Path Aliases

Use `@/` for imports from `src/` (configured in `tsconfig.json`).

## Code Style

- Biome for linting and formatting (tabs, double quotes)
- Run `npx biome check --write .` to format/lint
- UI text is in Korean

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/acee90) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
