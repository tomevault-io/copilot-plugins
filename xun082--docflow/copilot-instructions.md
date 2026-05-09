## 90-validation

> Validation workflow and allowed scripts (pnpm only)


Use pnpm for all scripts. The project is always running; do not use `pnpm dev`.

Before considering work complete, ensure all pass:

1. `pnpm type-check`
2. `pnpm lint`
3. `pnpm format`

If script names differ in package.json, update them accordingly so these commands exist and succeed.

---
> Source: [xun082/DocFlow](https://github.com/xun082/DocFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
