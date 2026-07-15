## sonit-portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands run from the `portfolio/` subdirectory:

```bash
cd portfolio
npm run dev          # dev server at http://localhost:3000
npm run build        # production build → dist/
npm run preview      # preview the production build at port 5173
npm run lint         # Prettier + ESLint + Stylelint + Markdownlint (all checks)
npm run lint:eslint:fix   # auto-fix ESLint issues
npm run lint:style:fix    # auto-fix Stylelint issues
npm run format       # auto-format with Prettier
npm run test:e2e     # Playwright end-to-end tests
npm run test:a11y    # axe-core accessibility scan
```

There is no unit test runner — testing is e2e (Playwright) and accessibility (axe-core) only.

## Architecture

Single-page React 18 + Vite 5 portfolio. No routing library — navigation is scroll-based using `react-scroll`. Tailwind CSS 3 for all styling with Framer Motion for animations.

### Source layout

```
portfolio/src/
  App.jsx              # root: loading gate → Navbar + section order + Footer
  data/                # content-only JS files (projects, experience, education, skills)
  components/
    Layout/            # Navbar, Footer, BackToTop
    Sections/          # Hero, About, Education, Projects, Experience, Skills
    UI/                # reusable primitives: ProjectCard, SkillCategory, TimelineItem, TechTag, Section
    utils/             # LoadingScreen
  hooks/
    useActionSection.js  # scroll-based active section detection
  styles/
    globals.css        # Tailwind directives + fluid typography CSS vars + focus styles
```

### Content data flow

Portfolio content lives entirely in `src/data/*.js` files as exported arrays — no CMS, no API. To update content (projects, experience, education, skills), edit those files directly.

Section components import from `data/` and render using UI primitives (`ProjectCard`, `TimelineItem`, `SkillCategory`, `TechTag`).

### Design tokens

Custom Tailwind tokens in `tailwind.config.js`:

- **Colors**: `primary` (#4F46E5 indigo), `dark.bg/bg2/bg3` (dark backgrounds), `glass.bg/border/highlight` (glassmorphism), `text.primary/secondary/tertiary`
- **Fonts**: `font-montserrat` (body), `font-space-grotesk` (headings)
- **Animations**: `animate-float` (6s vertical bob), `animate-shine` (gradient sweep)
- **Shadows**: `shadow-primary`, `shadow-glow`

### Active section tracking

`useActiveSection` hook (misnamed file: `useActionSection.js`) uses scroll position + `offsetTop` to highlight the current nav item. Section IDs are hardcoded in the hook: `['home', 'education', 'projects', 'experience', 'skills']`.

## Pre-commit hook

Husky runs `lint-staged` on every commit. The pre-commit hook also calls a bare `pre-commit` command — ensure that binary exists or remove the call if it causes issues.

## Deployment

Push to `main` triggers GitHub Actions CI/CD:

1. Lint → Build → Playwright e2e → axe a11y
2. Deploy (only if all pass): OIDC → temporary AWS creds → `aws s3 sync` → CloudFront invalidation

HTML files are synced with `Cache-Control: no-cache`; `/assets/*` (content-hashed by Vite) get `max-age=31536000, immutable`. Only `/index.html` and `/` are invalidated in CloudFront.

---
> Source: [sonit-ambashta1/sonit-portfolio](https://github.com/sonit-ambashta1/sonit-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
