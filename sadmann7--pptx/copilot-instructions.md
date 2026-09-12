## testing

> Testing conventions for workspace packages


# Testing Conventions

## Location

All tests and test fixtures live in `src/tests/` inside each package, never co-located next to source files.

```text
packages/core/src/tests/
  minimal-pptx.ts          # fixture builder (not a test)
  presets.test.ts
  text-renderer.test.ts
packages/react/src/tests/
  minimal-pptx.ts
  store.test.ts
  root.test.tsx
```

## Tooling

- Vitest with the `happy-dom` environment (the parser needs `DOMParser`; react tests need a DOM).
- Test deps (`vitest`, `happy-dom`, `@testing-library/react`) are versioned in the pnpm catalog (`pnpm-workspace.yaml`) and referenced as `"catalog:"` in each package's `devDependencies`, never installed in the root `package.json`.
- Each testable package has `"test": "vitest run"` and `"test:watch": "vitest"` scripts; `turbo test` runs everything.

## Style

- No mocks. Test real implementations with real inputs: build in-memory .pptx fixtures with the `minimal-pptx.ts` builders and run them through the actual parse/render pipeline (`parseZip` → `buildPresentation` → `renderSlide`).
- Prefer end-to-end assertions on real output (SVG path data, DOM structure, store state) over asserting internals.
- When fixing a rendering bug, add a regression test that fails on the old code; verify by temporarily reverting the fix.

---
> Source: [sadmann7/pptx](https://github.com/sadmann7/pptx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
