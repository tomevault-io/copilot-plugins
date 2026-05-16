## chiragch-com

> Personal portfolio website for Chirag Chandrashekar (Yashuchirag on GitHub).

# chiragch.com — Portfolio Project

## Project Overview
Personal portfolio website for Chirag Chandrashekar (Yashuchirag on GitHub).
Deployed on Netlify free tier at chiragch.com.

## Stack
- **Framework:** Vite + React 18 (migrating from Create React App)
- **Styling:** Tailwind CSS v4
- **Animation:** Framer Motion (replacing AOS)
- **Icons:** react-icons
- **Deployment:** Netlify free tier (static site, no SSR)

## Key Decisions & Preferences
- Single long-scroll page — no React Router, no fake page state switching
- Smooth anchor navigation with scroll-spy for active nav highlighting
- EmailJS integration is skipped — contact section uses mailto link + social links only
- No `@react-pdf/renderer` or `react-pdf` — resume is a static PDF in `public/`
- No axios — was only used for raw EmailJS HTTP calls which are now removed

## File Structure
```
src/
  main.jsx              # Vite entry
  App.jsx               # Layout shell only
  components/
    layout/             # Header, Footer
    sections/           # One file per page section
    ui/                 # Reusable primitives (Button, SectionHeading, etc.)
  data/                 # Content data files (do not add logic here)
  utils/                # Hooks and helpers
  styles/               # index.css with Tailwind directives + CSS vars only
public/
  chirag_1.jpg          # Profile photo
  Chirag_Resume.pdf     # Resume (linked directly, not rendered in-app)
```

## Content Data Files
- `src/data/Experience.js` — Work history (Glenysys, Accenture, CU Boulder TA)
- `src/data/Projects.js` — Projects (Netflix Clone, AI App, Volunteer Mgmt, etc.)
- `src/data/Skills.js` — Skill categories and tags
- `src/data/Education.js` — Education history

## Netlify Config
- Build command: `npm run build`
- Publish dir: `dist` (Vite output — NOT `build`)
- Node version: 20
- No env vars required (EmailJS removed)

## Code Style
- Use `.jsx` extension for all React components
- Functional components only, no class components
- Tailwind utility classes preferred over custom CSS
- Framer Motion `whileInView` + `viewport={{ once: true }}` for scroll animations
- Keep data files pure — no imports, no JSX, just exported arrays/objects

## Things to Avoid
- Do not add `useState` page routing (was the old broken pattern)
- Do not use AOS (`data-aos` attributes) — replaced by Framer Motion
- Do not use axios — removed dependency
- Do not add `REACT_APP_*` env vars — Vite uses `VITE_*` prefix
- Do not create new CSS files for individual components — use Tailwind
- Do not add progress bar skill charts — they look junior and are meaningless
- Do not mock EmailJS or add form submission without a real working service

## Deployment Notes
- Netlify free tier: 300 build minutes/month, 100GB bandwidth
- SPA redirect rule in `netlify.toml` handles direct URL visits
- Resume PDF served as static file from `public/` — no in-app PDF rendering

---
> Source: [Yashuchirag/chiragch.com](https://github.com/Yashuchirag/chiragch.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
