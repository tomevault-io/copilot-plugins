## weapp

> This repository builds VekUI WeApp: a shadcn-style source registry and first-party UI foundation for Taro React WeChat mini programs.

# Agent Guide

## Project Goal

This repository builds VekUI WeApp: a shadcn-style source registry and first-party UI foundation for Taro React WeChat mini programs.

## Required Reading Order

1. `README.md`
2. `docs/DEVELOPER_GUIDE.md`
3. `docs/UI_RULES.md`
4. `docs/TEST_PLAN.md`
5. `packages/registry/src/manifest.ts`

## Hard Rules

- Treat `packages/ui` as the canonical source for shared UI.
- Use `apps/miniprogram` only as a playground and runtime verification app.
- Use `packages/registry` as the public registry contract.
- Do not add third-party UI component libraries as a fallback.
- Do not import `@radix-ui/*`, `lucide-react`, browser DOM APIs, `ReactDOM`, `window`, or `document` inside `packages/ui`.
- UI styles must use semantic tokens and mini-program-safe Tailwind utilities.
- Mini program builds use the Taro Vite compiler.
- Every public UI component needs tests for API, state attributes, token classes, and mini-program compatibility rules.
- Keep developer docs and AI rules updated before large implementation changes.

## Verification

Run these checks before reporting completion:

```bash
pnpm typecheck
pnpm test
pnpm check:ui
pnpm registry:build
pnpm build:miniprogram
pnpm build:docs
```

---
> Source: [vekui/weapp](https://github.com/vekui/weapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
