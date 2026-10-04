## at-landing

> Quick context for AI agents working in this repo. Read `memory/` for details.

# AT Fitness Landing Page — Agent Memory

Quick context for AI agents working in this repo. Read `memory/` for details.

## What this is

A **bilingual (EN / AR) single-page fitness-coaching landing page** for "AT Fitness".
Adapted from an old "AirLens" photography-portfolio template — `README.md` now documents AT Fitness; `info.md` keeps the original template docs for reference (annotated at the top).

## Fast facts

- **Stack:** React 19 + TypeScript · Vite 7 · Tailwind CSS 3 · GSAP + ScrollTrigger · Lenis smooth scroll · three.js (hero 2.5D effect, dynamic import) · lucide-react icons
- **Content lives in config, not components:**
  - `src/config.ts` → English content + all TypeScript interfaces
  - `src/config.ar.ts` → Arabic content (same shapes)
  - Sections read copy via `useConfig()` (`src/hooks/useConfig.ts`)
- **Language** is toggled at runtime via `src/context/LanguageContext.tsx` (`atf-lang` in localStorage; **Arabic is the default for first-time visitors**). It sets `html.lang`, `html.dir`, and AR fonts. **When adding copy, always add it to both config files.**
- **Theme:** near-black `#09090B` background, amber `#F59E0B` accent, cyan `#22D3EE` secondary. Reusable classes in `src/App.css` (`.glass-card`, `.btn-primary`, `.btn-secondary`, `.eyebrow-badge`, `.section-title`, `.neon-line`, `.text-glow-amber`, `.savings-badge`, `.card-3d`).
- **Section order** (`src/App.tsx`): NeonOrbs → LanguageToggle → Hero → IntroGrid → Services → WhyChooseMe → FeaturedProjects *(renders the "Programs" section)* → Pricing → Testimonials → FAQ → Footer
- **Anchor IDs:** `#hero` `#work` `#services` `#why-choose-me` `#programs` `#pricing` `#testimonials` `#faq`
- **Commands:** `npm run dev` · `npm run build` (`tsc -b && vite build`) · `npm run lint` · `npm run preview`
- **Env:** Windows / PowerShell · path `D:\AT\Landing Page\app` · git repo (remote: `https://github.com/bedotaha1/AT-Landing.git`) · `.vercel/` present · `base: './'` in `vite.config.ts`

## Memory files

| File | Contents |
|---|---|
| `memory/00-index.md` | Index of memory files |
| `memory/01-project.md` | Overview, stack, scripts, folder map, assets, deployment |
| `memory/02-architecture.md` | Data flow, i18n system, hooks, config interfaces, conventions |
| `memory/03-design-system.md` | Colors, typography, CSS utilities, animation patterns, RTL rules |
| `memory/04-sections.md` | Per-section implementation breakdown |
| `memory/05-content.md` | Content reference (pricing, stats, FAQs, assets usage) |
| `memory/06-status-and-issues.md` | Known typos, inconsistencies, unused files, TODOs |

## Rules of thumb

1. Never hard-code user-facing copy in a component — put it in `config.ts` + `config.ar.ts`.
2. Keep every section's `null` guard (render nothing when config is empty).
3. GSAP animation classes follow the `.xxx-anim` convention + ScrollTrigger `once: true` + staggered delay.
4. Respect RTL: use the `isRTL` flag for alignment/order tweaks; CSS overrides live at the bottom of `src/App.css`.
5. Update `memory/` after significant changes so the next session starts fast.

---
> Source: [bedotaha1/AT-Landing](https://github.com/bedotaha1/AT-Landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
