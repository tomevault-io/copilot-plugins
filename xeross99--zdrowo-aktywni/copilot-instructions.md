## zdrowo-aktywni

> **Zdrowo Aktywni** — a Polish health & wellness website for Aleksandra Kost, hosted at https://zdrowo-aktywni.pl.

# CLAUDE.md

## Project Overview

**Zdrowo Aktywni** — a Polish health & wellness website for Aleksandra Kost, hosted at https://zdrowo-aktywni.pl.

## Tech Stack

- **Framework:** Next.js 15 (App Router)
- **Language:** JavaScript (JSX), no TypeScript
- **Styling:** Tailwind CSS v4 with PostCSS
- **UI Libraries:** Headless UI, Heroicons, Framer Motion, React Aria
- **Analytics:** Google Analytics (via @next/third-parties)

## Commands

- `npm run dev` — start dev server
- `npm run build` — production build
- `npm run lint` — run ESLint
- `npx prettier --check .` — check formatting

## Project Structure

```
src/
  app/
    (main)/page.jsx       — homepage
    kontakt/              — contact page
    o-mnie/               — about page
    sklep/                — shop (product pages)
    layout.jsx            — root layout
  components/             — shared components (Navbar, Footer, FadeIn, Icons, Container)
  styles/                 — Tailwind CSS
public/                   — static assets
```

## Conventions

- Language in UI: **Polish** — all user-facing text must be in Polish
- Components use `.jsx` extension
- Path aliases: `@/` maps to `src/`
- Formatting: Prettier with tailwindcss plugin
- Linting: ESLint with next config

---
> Source: [Xeross99/Zdrowo_aktywni](https://github.com/Xeross99/Zdrowo_aktywni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
