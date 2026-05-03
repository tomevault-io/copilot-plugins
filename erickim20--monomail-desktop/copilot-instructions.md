## design-rule

> **Design & Code Style Guidelines**


**Design & Code Style Guidelines**

1. **Use Semantic Color Tokens Only**
    Avoid raw utility colors (e.g., `bg-gray-200`, `text-red-500`).
    Use semantic tokens instead (e.g., `text-muted-foreground`, `bg-accent`).

2. **Avoid Explicit `type` Declarations**
    Do not use the `type` attribute unless it represents dynamic or contextual data
    (e.g., `type="audience_attribute"` is acceptable).

3. **Use `MonoIcon` Instead of `lucide-react`**

Do not import icons directly from `lucide-react`.
Use the `MonoIcon` component to ensure consistent sizing, styling, and customization across the app.

---
> Source: [erickim20/monomail-desktop](https://github.com/erickim20/monomail-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
