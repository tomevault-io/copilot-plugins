## rockylinux-org

> Read [docs/e2e/testing-patterns.md](../docs/e2e/testing-patterns.md) for Playwright patterns and gotchas specific to this project (Radix UI portaled components, accessible selectors, etc.).

# E2E Tests

## Before Writing Tests

Read [docs/e2e/testing-patterns.md](../docs/e2e/testing-patterns.md) for Playwright patterns and gotchas specific to this project (Radix UI portaled components, accessible selectors, etc.).

## Conventions

- Reusable selectors and shared page actions go in `e2e/utils/PageUtils.ts`; page-level assertion helpers go in `e2e/utils/PageAssertions.ts`; page- or feature-specific utilities (including download helpers such as `DownloadUtils.ts`) live alongside their respective page objects or flows
- Target components by accessible name (`getByRole` with `{ name }`) rather than generic roles or test IDs

---
> Source: [rocky-linux/rockylinux.org](https://github.com/rocky-linux/rockylinux.org) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
