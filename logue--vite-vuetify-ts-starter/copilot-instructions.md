## vite-vuetify-ts-starter

> Use when editing Vue UI, components, views, router wiring, or TypeScript code in src. Covers Vue 3 script-setup patterns, import alias usage, and minimal-change style for this template.


# Frontend Vue Rules

- Use Vue 3 SFC with `<script setup lang="ts">` for new or updated components.
- Keep route pages in `src/views` and reusable UI in `src/components`.
- Keep router changes centralized in `src/router.ts`.
- Use `@` alias for imports from `src`.
- Preserve strict TypeScript behavior; do not weaken tsconfig rules.
- Keep changes scoped and avoid broad refactors unless explicitly requested.
- Follow existing import grouping and rely on lint rules for final order.

## Validation

- For runtime-impacting changes, run `pnpm type-check`.
- Run relevant tests when behavior changes:
  - unit: `pnpm test:unit`
  - e2e: `pnpm test:e2e`

## References

- Project-wide defaults: `.github/copilot-instructions.md`
- Lint/import details: `eslint.config.ts`
- Build/chunking behavior: `vite.config.ts`

---
> Source: [logue/vite-vuetify-ts-starter](https://github.com/logue/vite-vuetify-ts-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
