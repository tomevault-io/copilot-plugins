## tools

> - **Language:** Vanilla TypeScript only. No frontend frameworks (React, Vue, Svelte, Angular, etc.).

# Claude Code — Monorepo Standards

## Tech Stack
- **Language:** Vanilla TypeScript only. No frontend frameworks (React, Vue, Svelte, Angular, etc.).
- **Package Manager & Runtime:** Bun. Use `bun install`, `bun run`, `bun test` for all operations.
- **Build:** Vite with zero-config Vanilla TS template. Production builds must have `minify: false` and `sourcemap: true`.
- **Testing:** Playwright for E2E and visual regression. All features must be developed using strict TDD — write a failing test before implementation code.
- **Database:** IndexedDB via the `idb` npm package. Offline-first architecture is mandatory.

## Development Rules
1. **TDD is mandatory.** Every feature starts with a failing Playwright test. No implementation code before a test exists.
2. **Offline-first.** All data lives in IndexedDB. The app must function without network connectivity.
3. **PWA required.** Service worker for caching and push notifications. Manifest for installability.
4. **Mobile-first CSS.** Design for phones first, then scale up with media queries.
5. **No frameworks.** Use vanilla DOM APIs, TypeScript, and CSS. Web Components are acceptable.
6. **Readable builds.** Never enable minification. Always generate sourcemaps.

## Project Layout
```
workout-tracker/    # Resistance training tracker PWA
  src/              # TypeScript source
  public/           # Static assets and PWA manifest
  e2e/              # Playwright E2E tests
  vite.config.ts    # Vite config
  playwright.config.ts
```

## Commands
```bash
cd workout-tracker && bun install          # Install deps
cd workout-tracker && bun run dev          # Dev server
cd workout-tracker && bun run build        # Production build
cd workout-tracker && bun run typecheck    # TypeScript checking
cd workout-tracker && bunx playwright test # Run E2E tests
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/willchan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/willchan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
