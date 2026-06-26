## jatinaneja-portfolio

> - Vue 3 + TypeScript + Vite

# AGENTS.md

## Stack
- Vue 3 + TypeScript + Vite
- Tailwind CSS + DaisyUI + @tailwindcss/typography
- vue-router, marked (Markdown rendering), Swiper/vue3-carousel, Phosphor Icons

## Commands
- `npm run dev` — hot-reload dev server (Vite)
- `npm run build` — production build to `dist/`
- `npm run preview` — serve local production build
- **No linting, no tests.** Nothing in package.json scripts besides dev/build/preview.

## Content Configuration
All site content is centralized in a single file — this is the primary edit target for any content-related change:
- `src/config/templateConfig.ts` — profile info, social links, timeline (experience/education), blog posts, ability items

## Deployment
Hosted on Azure Static Web Apps. `staticwebapp.config.json` in the repo root handles routing rules (SPA fallback).

---
> Source: [JatinAneja1812/jatinaneja-portfolio](https://github.com/JatinAneja1812/jatinaneja-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
