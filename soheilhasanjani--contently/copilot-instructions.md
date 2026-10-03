## components

> shadcn ui/common, Material Symbols icons, toasts


# Components

- shadcn CLI → `components/ui` (includes `cn` setup).
- Complex shared UI + `AppProviders` → `components/common`.
- **Icons**: Material Symbols Rounded — `@/components/common/icon` (`<Icon name="…" />`; typed allowlist in `ICONS`).
- **Toasts**: `@/components/ui/toast` — `<Toaster />` in `AppProviders`; `toast.add({ title, type })` (`success` | `error` | `info` | `warning` | `loading`).
- Prefer `next/image`, `next/font`, Next navigation. Dynamic-import heavy UIs.
- TipTap deferred until editor feature starts.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
