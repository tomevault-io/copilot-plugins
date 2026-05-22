## sscba-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React + TypeScript + Vite portfolio site for Shiv Chandekar. Uses Tailwind CSS v4, Framer Motion for animations, React Flow (@xyflow/react) for architecture diagrams, and Lucide icons.

## Development

**Run locally:**
```
npm run dev
```
Starts the Vite dev server with hot module replacement. Open `http://localhost:5173` in a browser.

**Build for production:**
```
npm run build
```
Runs `tsc -b && vite build`. Output goes to `dist/`.

**Preview production build:**
```
npm run preview
```

**Deploy to GitHub Pages:**
```
npm run deploy
```
Runs `predeploy` (build) then pushes `dist/` to the `gh-pages` branch via the `gh-pages` package.

**Path alias:** `@/` maps to `src/` (configured in `vite.config.ts`).

## Architecture

### File Structure
```
src/
  main.tsx                  # Entry point
  App.tsx                   # Root component, section layout
  index.css                 # Global styles + Tailwind imports
  components/
    effects/
      AnimatedBackground.tsx  # CSS dot-grid + floating data packets
      ScrollToTop.tsx
    layout/
      Navbar.tsx
      Footer.tsx
    projects/
      ProjectCard.tsx         # Project card with arch diagram toggle
      ReconDiagram.tsx        # React Flow diagram for Recon Engine
      HyperStreamDiagram.tsx  # React Flow diagram for HyperStream
      IntelliQueryDiagram.tsx # React Flow diagram for IntelliQuery
    sections/
      Hero.tsx
      About.tsx
      Skills.tsx
      Projects.tsx
      Contact.tsx
  data/
    projects.ts             # Project data (titles, tech, links, etc.)
    skills.ts               # Skills list
  hooks/
    useActiveSection.ts     # Intersection Observer for nav highlighting
    useCountUp.ts           # Animated number counter
    useTypingEffect.ts      # Typewriter effect for hero
  lib/
    utils.ts                # cn() helper (clsx + tailwind-merge)
```

### Theme & Design Tokens
Dark data-pipeline aesthetic. Key color values used throughout:
- Background: `#0a0f1e`, Surface cards: `#0d1b2e`
- Primary accent: `#00d4ff` (cyan), Secondary: `#10b981` (emerald)
- Muted text: `#94a3b8`, Primary text: `#e2e8f0`

### Architecture Diagrams
Each project card has a `[ View Architecture ]` toggle. On open, the relevant React Flow diagram component renders (`ReconDiagram`, `HyperStreamDiagram`, `IntelliQueryDiagram`). Nodes and edges are defined inline with custom styles matching the dark theme.

### Static Assets
`public/resources/profile.jpg` — profile photo
`public/resources/Shiv_Chandekar_Resume.pdf` — resume download

### Vite Config
`base: '/portfolio/'` — required for GitHub Pages deployment under a non-root repo path.

## Git Workflow
Work on the `react` branch (current active development). `main` is the production branch (GitHub Pages deployment).

---
> Source: [sscba/sscba.github.io](https://github.com/sscba/sscba.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
