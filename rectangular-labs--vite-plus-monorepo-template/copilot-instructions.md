## vite-plus-monorepo-template

> - Export shared icons for external consumers from `packages/ui/src/components/icon.tsx`.

# Repository Conventions

## UI

### Icons

- Export shared icons for external consumers from `packages/ui/src/components/icon.tsx`.
- When code outside the `@rectangular-labs/ui` package needs an icon, import it from `packages/ui/src/components/icon.tsx` rather than directly from `@phosphor-icons/react`.
- Code inside the UI package may import directly from `@phosphor-icons/react` when that is the most direct internal implementation detail.

Examples:

- `packages/ui/src/components/core/button.tsx` may use `@phosphor-icons/react` directly.
- `apps/demo-app/src/routes/login/-shared.tsx` should consume icons from `packages/ui/src/components/icon.tsx`.

---
> Source: [rectangular-labs/vite-plus-monorepo-template](https://github.com/rectangular-labs/vite-plus-monorepo-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
