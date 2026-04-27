## sourav27-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for **Sourav Debnath** (single-page). For any content work, always review `docs/context/sourav-resume-2026.md` for accurate personal details, experience, education, skills, and awards. built with React 19, TypeScript, Vite, and Tailwind CSS v4. Deployed to GitHub Pages via GitHub Actions. **Design is intentionally in flux** — the visual direction is still being explored, so keep implementations flexible and avoid hardcoding aesthetic decisions.

## Development Commands

```bash
npm run dev      # Start development server
npm run build    # Type check and build for production
npm run lint     # Run ESLint
npm run preview  # Preview production build locally
```

## Architecture

### Tech Stack
- **Framework**: React 19 with TypeScript
- **Styling**: Tailwind CSS v4 (via `@tailwindcss/vite` plugin — not PostCSS)
- **Animations**: Framer Motion
- **Routing**: React Router v7 (anchor-based navigation only — no multi-page routes)
- **Icons**: Lucide React
- **Utilities**: `clsx` + `tailwind-merge` for conditional classes

### Project Structure

```
src/
├── pages/Home.tsx          # Single page — composes all sections
├── components/             # One component per section (Hero, About, VisualResume, Gallery, etc.)
├── data/                   # All content lives here as typed TS exports
│   ├── experiences.ts      # ExperienceItem[] — work history for VisualResume
│   └── gallery.ts          # (planned) GalleryItem[] — photo gallery content
├── App.tsx                 # Router, global dark theme, Navbar, Footer
└── index.css               # Tailwind directives only
```

### Navigation
This is a **single-page app with anchor-based navigation**. Sections use `id` attributes (`id="about"`, `id="resume"`, `id="gallery"`). No React Router `<Link>` to other routes — use `href="#section-id"` for in-page nav. Multi-page routing on GitHub Pages requires a `404.html` redirect hack; avoid unless necessary.

### Content Updates
All portfolio content is data-driven via `src/data/`. To update content:
- **Work history**: Edit `src/data/experiences.ts` — `ExperienceItem` interface is the source of truth.
- **Gallery**: Edit `src/data/gallery.ts` (being migrated from hardcoded `Gallery.tsx`).
- Do **not** hardcode content in component files.

### Styling Conventions
- Dark theme: `bg-stone-950` (deepest) / `bg-stone-900` (section alternates) / `text-stone-100`.
- Accent: orange palette (`orange-500`, `orange-600`). Selection highlight: `selection:bg-orange-500 selection:text-white` set globally in `App.tsx`.
- `mix-blend-*` modes are used in `Hero.tsx` for layering effects — be careful when adding overlays near that component.
- `cursor-none` is set on the Hero container; a custom motion cursor is rendered via Framer Motion. Don't apply `cursor-*` utilities inside Hero.

### Framer Motion Conventions
- Scroll-triggered animations use `whileInView` + `viewport={{ once: true }}` — this is the standard pattern.
- Staggered children use `transition={{ delay: index * 0.1 }}`.
- Known issue: **animations can conflict or not trigger** if the parent doesn't have enough height, or if `viewport` margin is too aggressive. If an animation doesn't fire, check `margin` on `viewport` prop (e.g. `margin: "-100px"`).
- `useMotionValue` is used in `Hero.tsx` for mouse tracking — avoid `useState` for high-frequency mouse events.

### Deployment
- Push to `main` → GitHub Actions builds and deploys to GitHub Pages automatically.
- Uses Node 20 in CI (`npm ci` then `npm run build`).
- `dist/` is the build output; never commit it manually.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sourav27) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
