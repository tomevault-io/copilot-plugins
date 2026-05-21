## tailwindcss-v4

> When you want to use the tailwindcss on the client side


# Tailwind CSS v4 Guide

- Use utility-first classes for consistent and maintainable styling
- Create custom components with `@apply` directive for reusability
- Utilize responsive design utilities for mobile-first development
- Implement dark mode support using Tailwind's built-in classes


- For the tailwind css v4 we are using `@tailwindcss/postcss` and `postcss.config.mjs` PostCSS configuration.

```mjs

const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;

```

- This is the current globals.css file [globals.css](mdc:app/globals.css).

---
> Source: [VidAIze/cursor-project-rules](https://github.com/VidAIze/cursor-project-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
