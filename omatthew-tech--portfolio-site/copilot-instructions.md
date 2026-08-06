## portfolio-site

> - Stack: Astro 5 (static) + TS strict + MDX + Tailwind v4 + Cloudflare Pages + Cloudflare Images

# AGENTS.md

- Stack: Astro 5 (static) + TS strict + MDX + Tailwind v4 + Cloudflare Pages + Cloudflare Images
- Single source of design truth: DESIGN.md (Google DESIGN.md format). Never hardcode colors/fonts/spacing; use exported Tailwind tokens.
- Commands: `pnpm dev | build | preview | typecheck | tokens | design:lint | test | lh`
- Never add React/Vue/Svelte. Islands only on real interactivity need.
- A11y: WCAG 2.2 AA, 44px touch min, full keyboard path, `prefers-reduced-motion` respected.
- Perf budgets: home <= 250 KB, case <= 600 KB (sans hero); LCP <= 1.5s mobile.
- Content lives in `src/content/work` as MDX with Zod-validated frontmatter.
- Images: Cloudflare Images via `<Figure>`; never raw `<img>` for case-study assets.
- Definition of done: all gates in PLAN.md section 7 green.

---
> Source: [omatthew-tech/Portfolio-Site](https://github.com/omatthew-tech/Portfolio-Site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
