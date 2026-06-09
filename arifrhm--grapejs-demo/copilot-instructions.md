## grapejs-demo

> **Critical:** All GrapesJS editor components must be loaded with `ssr: false` in Next.js using `next/dynamic`:

# AGENTS.md

## GrapesJS Integration Constraint

**Critical:** All GrapesJS editor components must be loaded with `ssr: false` in Next.js using `next/dynamic`:

```tsx
const GrapesEditor = dynamic(
  () => import('@/components/GrapesEditor'),
  { ssr: false }
);
```

GrapesJS depends on browser-only APIs (`window`, `document`) and will fail with "window is not defined" error if rendered server-side.

All editor components must be in files with `"use client"` directive.

---
> Source: [arifrhm/grapejs-demo](https://github.com/arifrhm/grapejs-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
