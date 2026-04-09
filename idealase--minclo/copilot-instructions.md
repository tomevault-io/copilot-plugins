## minclo

> - **Build**: `npm run build` (`tsc -b && vite build`)

# minclo

## Quick Reference
- **Build**: `npm run build` (`tsc -b && vite build`)
- **Test**: `npm test` (Vitest, single run)
- **Test watch**: `npm run test:watch`
- **Test coverage**: `npm run test:coverage` (V8 coverage)
- **Lint**: `npm run lint` (ESLint)
- **Format**: `npm run format` (Prettier)
- **Dev server**: `npm run dev` → http://localhost:5173
- **Deploy**: Push to `main` or `master` → GitHub Actions builds and deploys to GitHub Pages

## Architecture
Interactive mine closure cost estimation tool with real-time parametric calculations.

```
/src/
  /domain/              → Pure calculation engine + Zod validation (calcEngine.ts, validation.ts)
    *.test.ts           → Co-located unit tests
  /charts/              → D3.js chart modules (Breakdown, Cashflow, Tornado sensitivity)
  /components/          → InputPanel, OutputPanel (CSS Modules)
  /ui/                  → Reusable UI components (SliderInput, KPICard, CollapsibleSection, etc.)
  /state/               → React Context (ThemeContext) + store.tsx
  /utils/               → Formatting helpers, export (JSON/CSV)
  /main.tsx             → Entry point
```

## Key Conventions
- **Branch**: `master` (not `main`) — but CI triggers on both `main` and `master`
- **D3 charts**: Direct DOM manipulation via D3 (not React-rendered SVG)
- **Zod 4**: Runtime validation of all input parameters
- **CSS Modules**: `*.module.css` for component styles (not Tailwind, not styled-components)
- **Path alias**: `@/` → `src/` (configured in vite.config.ts and tsconfig)
- **Chunk splitting**: D3 and React in separate vendor chunks (vite.config.ts `manualChunks`)
- **Presets**: Predefined scenarios in `domain/presets.ts`
- **Prettier**: Enforced — semi, singleQuote, tabWidth 2, printWidth 100

## Deployment
- **URL**: https://idealase.github.io/minclo/
- **Base path**: `/minclo/` (set in vite.config.ts)
- **CI**: `deploy.yml` — Node 20, lint (continue-on-error) → test → build → gh-pages
- **Branch deploy**: peaceiris/actions-gh-pages to `gh-pages` branch

## Common Pitfalls
- Base path `/minclo/` must be set for GitHub Pages — asset URLs break without it
- D3 transitions can conflict with React re-renders — use refs for D3 containers
- Domain logic is pure TypeScript — no React imports in `/domain/` or `/charts/`
- Tests are co-located (`*.test.ts` next to source) — not in a separate `__tests__/` dir

## Existing Agent Guidance
See `.github/copilot-instructions.md` for D3 visualization patterns, architecture decisions, and agent scope rules.

## Sensitive Files
Do not read, log, or commit: any `.env` files, credentials, secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idealase)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/idealase)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
