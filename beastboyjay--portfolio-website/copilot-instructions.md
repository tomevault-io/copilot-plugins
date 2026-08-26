## portfolio-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page personal portfolio site for Jay Sinha (AI Engineer), built with Next.js 15 (App Router), React 19, TypeScript, and Tailwind CSS v4. Deployed statically as a marketing/portfolio page — no backend, no database, no API routes.

## Commands

```bash
npm run dev      # start dev server (Turbopack) at localhost:3000
npm run build    # production build
npm run start    # serve production build
npm run lint     # eslint
```

There is no test suite configured in this repo.

## Architecture

**Almost the entire site lives in one file:** `src/app/page.tsx` (~660 lines). It is a client component (`"use client"`) composed of section components defined and called in sequence at the bottom in `HomePage()`: `Navbar`, `HeroSection`, `TechMarquee`, `NowSection`, `WorkSection`, `ExperienceSection`, `WritingSection`, `SkillsSection`, `ContactSection`, `Footer`. Each section is self-contained JSX with inline Tailwind classes — there is no section-per-file split.

All page content (nav links, tech list, project list, experience, skills, stats) is defined as plain data arrays/objects at the top of `page.tsx` (e.g. `nav`, `tech`, `projects`, `experience`, `skills`, `nowCards`). To update site content (add a project, change a skill, edit copy), edit these arrays directly rather than touching JSX — the render logic below them is generic over the data shape.

Animation: Framer Motion (`motion`) drives scroll-reveal via a shared `reveal` transition preset and per-element `initial`/`animate` props. Custom CSS keyframe animations (marquee scroll, pulse dot, blinking cursor) are defined in `src/app/globals.css` and applied via utility classes (`animate-marquee`, `animate-pulse-dot`, `animate-blink-cursor`) — these are not Framer Motion.

`src/app/layout.tsx` sets up three Google fonts (Instrument Serif for display/serif headings, Hanken Grotesk for body, JetBrains Mono for code/mono UI) as CSS variables consumed by Tailwind's `font-serif`/`font-sans`/`font-mono` (wired in `globals.css` `@theme inline`), plus a JSON-LD `Person` schema block for SEO.

**shadcn/ui**: `src/components/ui/` contains the full shadcn "new-york" component set (`components.json`), but `page.tsx` does not currently use any of them — it hand-rolls all UI with raw Tailwind. Treat these as available building blocks for new work, not as in-use dependencies. Add new shadcn components via the registries declared in `components.json` (includes a custom `@react-bits` registry at `reactbits.dev`).

**Styling**: Tailwind v4 (CSS-first config, no `tailwind.config.js` — theme tokens live in `globals.css` under `@theme inline`). Colors are OKLCH throughout, including many one-off inline `oklch(...)` values in `page.tsx` rather than theme tokens — this is the existing convention for section-specific styling, not an oversight to "fix" opportunistically. `cn()` (`src/lib/utils.ts`) wraps `clsx` + `tailwind-merge` for conditional class composition.

**orchids-visual-edits**: The build is wired for a visual-editing tool (`next.config.ts` registers its loader for `*.jsx`/`*.tsx` files via Turbopack; `layout.tsx` mounts `<VisualEditsMessenger />`). Don't remove this wiring without checking if it's still needed for the deployment/editing workflow.

**Known rough edges**:
- `next.config.ts` sets `typescript.ignoreBuildErrors: true` and `eslint.ignoreDuringBuilds: true` — the production build will succeed even with type errors or lint failures. Run `tsc --noEmit` / `npm run lint` manually to catch issues.
- `tsconfig.json` explicitly includes `src/components/MagicBento.jsx`, which does not exist in the repo — this is stale config, not a missing file to recreate.

---
> Source: [BEASTBOYJAY/Portfolio-Website](https://github.com/BEASTBOYJAY/Portfolio-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
