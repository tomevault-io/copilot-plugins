## munch

> This rule provides standards for frontend components, ensuring consistent styling and animations:

This rule provides standards for frontend components, ensuring consistent styling and animations:

When working in with React components:

- Always use `@radix-ui` components for low-level visual components
- Always use `@radix-ui/icons` components for icons

When working with routing:

- Follow `@tanstack/react-router` convensions and best practices
- Keep the routes typesaf

When adding imports:

- Order imports based on the following categories:
    1. External libraries (`react`, `@tanstack/react-router`, `sonner`, etc.)
    2. Assets (`@/assets`)
    3. Services (`@/services`)
    4. Components (`@/components`)
    5. Utils (`@/utils`)
- Within each category, keep related imports close - for example `@radix-ui/theme` should be close to `@radix-ui/icons`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomaszgil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomaszgil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
