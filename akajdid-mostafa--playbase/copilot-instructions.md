## playbase

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->

<!-- BEGIN:project-design-system-rules -->

# Project design system rules

- This project already uses shadcn. Use the default shadcn design system style, components, tokens, and usage patterns.
- Prefer components from `components/ui` and add new UI primitives through the shadcn CLI/registry instead of hand-rolling design-system components.
- Style screens with Tailwind utilities that reference the existing design tokens (`bg-background`, `text-foreground`, `text-muted-foreground`, `border-border`, `bg-card`, `bg-primary`, etc.).
- Image/media cards must use the configured shadcn `Card` style and design tokens. Do not copy reference-image corner treatments or hand-roll media wrappers; respect the local Card radius, border/ring, padding, and overflow behavior unless the user explicitly asks to change the design system.
- Do not create custom project styling in `app/globals.css`. Keep global CSS limited to the existing Tailwind/shadcn imports, token mappings, CSS variables, and base layer required by the design system.
- Do not replace or override the configured shadcn theme unless the user explicitly asks for a design-system change.
<!-- END:project-design-system-rules -->

<!-- BEGIN:project-animation-rules -->

# Project animation rules

- When the user asks to add animation, use the `motion` library from Motion (`motion/react`).
- Prefer reusable Motion helpers/patterns so timing, easing, viewport behavior, and stagger feel consistent across pages.
- Apply scroll reveal animations granularly to the smallest meaningful UI pieces, not only the parent section. Examples:
  - FAQ sections should animate each FAQ item one by one.
  - Two-column layouts should animate both left and right columns/content independently.
  - Card grids, image groups, stats, footer columns, forms, buttons, and text blocks should reveal as separate children with stagger when appropriate.
- For first-viewport hero sections, use load/mount animation (`initial` + `animate`) because the hero is visible by default, rather than scroll-triggered animation.
- Keep animations subtle: slide/fade motion, restrained duration, and no layout-shifting effects.
<!-- END:project-animation-rules -->

---
> Source: [akajdid-mostafa/playbase](https://github.com/akajdid-mostafa/playbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
