## cf-ai-contribot

> Enforce functional React components with hooks, no class components. Use tanstack-router for frontend routing and maintain route structure. separate shared UI, page-level views, helpers, and global layout. Use shadcn UI library with TailwindCSS, following semantic color tokens.

frontend.components.functionalComponents: true
frontend.codingStyle.react.rules:
  - Use functional components + hooks
  - Split components >150 lines
  - Avoid unnecessary state
  - Type-safe network requests via tanstack-query
frontend.router: tanstack-router
codingStyle.react.componentStructure:
  - components/ -> shared UI
  - routes/ -> page-level views
  - lib/ -> helpers and API clients
  - core/ -> global providers and layout
  - frontend.ui.library: shadcn
frontend.ui.styling: tailwind
frontend.ui.classConventions:
  - Semantic classes (text-primary, bg-muted, border-border)
  - Inline Tailwind must follow shadcn conventions
  - Avoid hard-coded colors or pixel values

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChadRosseau) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
