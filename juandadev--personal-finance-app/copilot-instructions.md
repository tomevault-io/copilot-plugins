## tailwind-theme-tokens

> Use Tailwind theme tokens for design-system colors


# Tailwind Theme Tokens

- Declare design-system CSS variables in `@theme` using Tailwind v4 token names like `--color-chart-1`, not legacy aliases in `:root` such as `--chart-1`.
- New reusable colors must be added to `app/globals.css` as `--color-*` tokens so Tailwind can generate utilities like `bg-*`, `text-*`, and `border-*`.
- Keep data and domain models token-based. Store values like `"chart-1"` or `"finance-purple"` instead of hex colors or `var(--color-*)` strings.
- Prefer Tailwind utility classes generated from tokens. For dynamic token usage, route through `lib/theme-colors.ts` instead of constructing arbitrary classes or inline styles in components.
- Only use `var(--color-*)` at integration boundaries that require a CSS color value, such as chart libraries that receive a `fill` prop.

---
> Source: [juandadev/personal-finance-app](https://github.com/juandadev/personal-finance-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
