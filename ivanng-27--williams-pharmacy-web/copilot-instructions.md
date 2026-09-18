## williams-pharmacy-web

> Next.js 15 (App Router) · TypeScript strict · Tailwind CSS · deploy on Vercel.

# CLAUDE.md — William's Pharmacy website

## Stack

Next.js 15 (App Router) · TypeScript strict · Tailwind CSS · deploy on Vercel.

## Conventions

- English only: code, comments, commits, docs.
- Minimal, readable code. No premature abstractions. 3 repeated lines > 1 early abstraction.
- Edit partially, never rewrite whole files.
- Products accessed only via `getProducts()` in `src/data/products.ts`.
- Copy lives in `src/content/*`, prefixed `PLACEHOLDER_`. Images are local placeholders.
- Accessibility WCAG 2.2 AA. Semantic HTML, visible focus, alt text.
- Respect `prefers-reduced-motion` in every animation.

## Design tokens

Navy (primary) base #152442 · Gold (accent) base #C4A157.
Gold is for accents/large headings only — never body text on white (contrast).
Fonts: Fraunces (display) + Inter (body), self-hosted via next/font.

## Do not

- No purple gradients, Bootstrap look, emoji icons, heavy shadows, lorem ipsum in final copy.
- No i18n library yet (English only). Keep copy centralized so it's easy to add later.

---
> Source: [IvanNG-27/Williams-Pharmacy-WEB](https://github.com/IvanNG-27/Williams-Pharmacy-WEB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
