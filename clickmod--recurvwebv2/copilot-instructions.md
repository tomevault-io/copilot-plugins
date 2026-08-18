## recurvwebv2

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

<!-- BEGIN:mobile-responsive-rules -->
# Mobile-First Responsive Design — Four Hard Rules

1. **Layout → Tailwind classes only.** `display`, `grid-template-columns`, `padding`, `gap`, `margin`, `width`, `font-size` for headings — all Tailwind.
2. **Tokens → inline style only.** Colors, `font-family`, `letter-spacing`, `border-color`, and specific brand values from `theme.ts`.
3. **Wrap every section in `<Container>`.** Never hardcode `padding: "0 56px"` or any fixed horizontal gutter.
4. **Typography via `var(--fs-*)` CSS custom properties.** No hardcoded pixel font sizes for headings.

See `.cursor/rules/mobile-responsive.mdc` for the full convention.
<!-- END:mobile-responsive-rules -->

<!-- BEGIN:section-component-rules -->
# Section-as-Component Architecture

Every distinct visual section on a page lives in its own component file:
- Home sections → `components/home/`
- Contact sections → `components/contact/`
- Shared across pages → `components/sections/`

Page files (`app/**/page.tsx`) import and compose these components. They contain no layout JSX directly.
<!-- END:section-component-rules -->

---
> Source: [ClickMod/RecurvwebV2](https://github.com/ClickMod/RecurvwebV2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
