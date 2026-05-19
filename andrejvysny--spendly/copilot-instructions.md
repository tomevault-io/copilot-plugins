## ui-shadcn

> Use when building or changing UI. Reuse shadcn/ui primitives and follow design system.


# UI / shadcn

- Prefer delegating to the `frontend` subagent for shadcn/Tailwind UI work.
- Reuse primitives from `resources/js/components/ui`. Do not duplicate; compose or extend.
- New components: build from ui (Button, Card, Dialog, Input, etc.); use `cn()` and CVA where appropriate.
- Icons: lucide-react.
- Reference: components.json (aliases), resources/js/components/ui/button.tsx (CVA + cn pattern).

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
