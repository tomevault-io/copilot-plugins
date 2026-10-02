## the-element

> This project is an AI-assisted, vibe-coded control surface for TouchDesigner. When modifying code, please preserve the following conventions:

# Agent / Contributor Notes

This project is an AI-assisted, vibe-coded control surface for TouchDesigner. When modifying code, please preserve the following conventions:

- **TanStack Start v1** — routes live in `src/routes/`. Do not add `react-router-dom` or Next.js-style pages.
- **Tailwind CSS v4** — use semantic design tokens from `src/styles.css`. Avoid hardcoded colors like `text-white` or arbitrary hex values.
- **Server functions** — use `createServerFn` from `@tanstack/react-start` for app-internal logic. External APIs/webhooks go under `src/routes/api/public/`.
- **WebSocket protocol** — TouchDesigner owns the source of truth for parameter values; the frontend owns layout/visual overrides. Do not overwrite user customizations on `ui_create` for existing IDs.
- **Components** — keep widgets small and composable. Reuse `CanvasWidget`, `CanvasShape`, `ContextualGroup`, and `CanvasGroupBox` patterns when adding new canvas elements.

Avoid force-pushing or rewriting published git history once the repo is public.

---
> Source: [OkamirufuVizualizer/the-element](https://github.com/OkamirufuVizualizer/the-element) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
