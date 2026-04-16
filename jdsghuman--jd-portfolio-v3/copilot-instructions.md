## jd-portfolio-v3

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server at localhost:3000
npm run build    # Production build
npm start        # Start production server
npm run lint     # Run ESLint
```

No test runner is configured.

## Architecture

Next.js 13 (Pages Router) portfolio site with TypeScript and SCSS Modules.

**Path aliases** (configured in `tsconfig.json`):
- `@components/*` → `src/components/*`
- `@styles/*` → `src/styles/*`
- `@data/*` → `src/data/*`

### Data Layer

All project data lives in `src/data/data.ts` as a typed array. Types are in `src/data/types.ts`. Adding a new project means adding an entry to this array and creating a new page under `src/pages/project/<slug>/index.tsx`.

### Pages & Routing

- `src/pages/index.tsx` — Home page, assembles all sections
- `src/pages/project/[name]/index.tsx` — One file per project (not dynamic routes)
- `src/pages/api/send-mail.ts` — Nodemailer/Zoho SMTP endpoint consumed by the Contact form via Axios
- `src/pages/resume/` — Resume page

### State Management

Two React Contexts in `src/store/`:
- `link-context.tsx` — Tracks the active nav section (used for nav highlight)
- `isTop-context.tsx` — Tracks whether the page is scrolled to top (controls nav styling)

Both are provided at the app level in `src/pages/_app.tsx`.

### Layout

`src/components/Layout/` wraps every page and renders Nav, SideDrawer, Backdrop, and Footer. The SideDrawer and Backdrop are toggled via state in Layout and connected to the mobile hamburger in Nav.

### Styling

SCSS Modules per component plus shared partials in `src/styles/`:
- `_variables.scss` — shadows, border rule
- `_color-palette.scss` — color variables
- `_breakpoints.scss` — responsive breakpoints
- `_animations.scss` — keyframe animations

Use `classnames` for conditional class composition.

### Images

External images must be allowlisted in `next.config.js` under `images.domains`. Currently `i.imgur.com` and `images.unsplash.com` are allowed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdsghuman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
