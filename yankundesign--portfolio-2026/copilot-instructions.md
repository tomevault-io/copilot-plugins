## portfolio-2026

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project overview

Personal portfolio for Yankun, a senior product designer. The site is built around a **desk-and-notebook** metaphor with three layers:

1. **Desk scene** (homepage) — an angled, perspective view of a workspace surface with physical objects: a notebook (→ works), a folded paper (→ resume), and a personal object (→ about). Objects are draggable. The desk *is* the navigation — no nav bar.
2. **Canvas / works** (works) — opening the notebook reveals a 12-column editorial plate where projects sit as hand-composed plates with museum-label captions. Same plate system as the desk's 8-col grid, denser column count. Static asymmetry comes from column-span variation, aspect-ratio variation, and vertical baseline alignment within row tracks. Rotation is available as an interaction and compositional tool when it earns its place (e.g., hover enlarge, peeking artifacts). Vertically scrollable spread.
3. **Project detail** (case study) — clicking a project cluster zooms in and opens a vertical-scroll editorial layout. Long-form, well-paced, freed from the two-page spread constraint.

The portfolio itself is a product demonstration.

**Design evolution:** Field Notebook was selected over Liner Notes. The paginated book format was then replaced with the desk + canvas + vertical scroll model to make the experience more interactive and less like a digital magazine. See `00-brief/decisions-log.md` and `00-brief/prd-homepage-desk.md` / `00-brief/prd-notebook-canvas.md` for full rationale.

The Field Notebook prototype (in `03-prototype-explore/`) established the visual language. That language (colors, typography, editorial plates, grain) is locked. The display format has evolved beyond it.

## Design source of truth

`00-brief/vibe-guideline.md` is the canonical reference for all visual and voice decisions. Consult it before making any design, styling, or copy decisions.

Key constraints from the guideline:
- Two-color palette only: ink blue (`--ink: #16265e`) on warm off-white (`--paper: #f4f1ea`)
- Typography: Fraunces (display + body), JetBrains Mono (metadata), Caveat (marginalia placeholder)
- No CSS frameworks, no Tailwind, no component libraries — plain CSS with design tokens is part of the craft demonstration
- Motion is weighted and paper-like. Respect `prefers-reduced-motion`. No scroll-jacking. The canvas has shallow spatial depth (not parallax in the scroll-hijacking sense)
- 12-column grid (1280px max, 24px gutter, 60px margin) for canvas and project detail pages. Desk uses an 8-col plate grid. The plate chrome (column rulings, plate metadata, register marks, fig captions) is shared across all surfaces — same system, different column counts.

## Repository structure

Design assets and planning live at the root. The deployable site lives in `site/`.

- `00-brief/` — Design briefs, vibe guideline, architecture plan (not deployed)
- `01-content/` — Raw case study writing as markdown drafts (not deployed)
- `02-assets/` — Design assets: illustrations, exports, source files (not deployed)
- `03-prototype-explore/` — Early HTML prototype explorations (not deployed, reference only)
- `04-reference/` — Visual references: Mary Kim zine, Jackie Zhang, type specimen (not deployed)
- `site/` — The Vite + React project (deployable)

## Build and development

The site uses **Vite + React + TypeScript** as a single-page application.

```bash
cd site
npm install
npm run dev          # Local dev server (Vite HMR)
npm run build        # Production build to dist/
npm run preview      # Preview production build
```

TypeScript strict mode for all code. Self-hosted fonts (woff2) with `font-display: swap`.

## Deploy

Live at **https://yankun.design** (also `https://yankun-portfolio.pages.dev`). Hosted on Cloudflare Pages, project `yankun-portfolio`. DNS is at Cloudflare.

**Deploys are manual.** Cloudflare's dashboard "Connect to Git" flow has a known OAuth pairing bug on this account — the GitHub App installs cleanly, but Cloudflare's callback fails to pair, so there is no auto-deploy on `git push`. Every deploy goes through Wrangler from local.

### Workflow

After making changes, from the repo root:

```bash
cd site
npm run build
npx wrangler pages deploy dist --project-name=yankun-portfolio --branch=main --commit-dirty=true
cd ..
git add .
git commit -m "<your message>"
git push
```

Build + Wrangler ships the live site. `git push` updates the GitHub repo. They are independent — running one does not trigger the other. Order is not strict, but deploying before pushing is the cleaner habit: a failed build gets fixed before a commit lands.

### Wrangler auth

First-time use opens a browser OAuth flow to Cloudflare; stays logged in ~30 days. If `wrangler` reports `Failed to fetch auth token`, run the command again — it retries via OAuth automatically.

### Future fix

Retry the dashboard git-connect flow occasionally — the OAuth bug tends to clear after several days. Pages → yankun-portfolio → Settings → Builds & deployments → "Connect to Git." If it succeeds, auto-deploy on push starts working and the Wrangler step becomes optional. If it errors, a GitHub Actions workflow with a Cloudflare API token is the backup. Decisions captured in `00-brief/decisions-log.md` under the 2026-05-10 entry.

## Image assets

Images in `site/public/` use two formats depending on usage:

**WebP at q=85** for desk-scene chrome, plates, cats, transitions, and the canvas notebook spread — 90% smaller than equivalent PNGs at indistinguishable quality. These load on every page, so weight matters.

**PNG** for `canvas/mockups/*` (project tiles on the canvas) and `images/chai/*` (CHAI case study detail screenshots) — kept lossless so text and fine UI remain crisp at zoom.

Original PNGs for WebP-converted images are still kept in `site/public/` alongside the WebPs for revertability. Code references the WebP. Reverting a single image is a one-line `.webp` → `.png` swap in the source — the original PNG is already there to serve.

When adding a new image:
- Mockup tile, case study screenshot, anything where pixel fidelity at zoom matters → PNG
- Anything else (textures, plates, illustrations, hero objects) → WebP at q=85

## Architecture (site/src/)

Per `00-brief/architecture-plan.md`:

- `styles/` — `tokens.css` (design tokens), `reset.css`, `typography.css`, `global.css`
- `components/desk/` — Desk scene: `DeskScene.tsx` (3D perspective container), `NotebookCover.tsx`, `ResumePaper.tsx`, `AboutObject.tsx`, `DeskSurface.tsx`
- `components/canvas/` — 12-col editorial plate (works): `Canvas.tsx` (grid container with intro paragraph + section captions), `ProjectCard.tsx` (column-spanned plate + museum-label caption), `MetaCard.tsx` (currently + colophon, fixed bottom-right)
- `components/project/` — Project detail (vertical scroll): `ProjectHeader.tsx`, `ProjectBody.tsx`, `MetricDisplay.tsx`, `PullQuote.tsx`, `Credits.tsx`
- `components/shared/` — Reusable: `EditorialPlate.tsx`, `Marginalia.tsx`, `StickyNote.tsx`, `Grain.tsx`, `Transition.tsx`
- `interactions/` — React hooks: `useDraggable.ts`, `usePanCanvas.ts`, `useDeskToCanvas.ts`, `useCanvasToProject.ts`, `useImageOverlay.ts`, `useReducedMotion.ts`
- `routes/` — Route components that wire together components and data
- `data/projects.ts` — Project metadata (titles, dates, credits, cluster positions)
- CSS Modules (`*.module.css`) for component-scoped styles

Components organized by concern: `desk/` for the homepage scene, `canvas/` for the works pinboard, `project/` for case study detail, `shared/` for reusable pieces. Interactions are React hooks.

## v1 scope

**In:** Desk scene homepage (notebook object functional, resume + about as placeholders), canvas (12-col editorial plate) with four project plates — CHAI, Control Hub Agentic, Write-like-Webex, SAP Fieldglass — full CHAI case study (vertical scroll), stubbed case studies for the other three, two journal pages, colophon, 404 page.

Note: Smart Search was retired from the canvas in v0.7. It remains a *proof beat inside* the CHAI case study (`smartSearch` in `chaiContent.ts`); it is no longer a top-level project.

**Out:** Essays/writing section, side projects, contact form (email link only), dark mode, resume page content, about-me page content.

## Hard rules

- No placeholder lorem ipsum — stub with real titles and short real paragraphs, or flag as needing content
- No AI-generated prose for case studies — content in `01-content/` is written by Yankun
- Screenshots are full-color editorial plates (1px ink border, mono caption below) — never duotoned, never full-bleed
- Handwriting font (Caveat) is a v1 placeholder — will be replaced with real handwriting before ship
- Marginalia is 2–6 words max, never more than 2 per screen
- The desk-to-canvas portal transition is the signature moment — invest craft here
- Canvas cards live on a 12-col editorial plate. Positions are hand-composed via column spans + row tracks in `data/projects.ts`, never computed by a layout algorithm. No off-grid placement at rest.
- The canvas's static compositional asymmetry comes from column-span variation, aspect-ratio variation, and vertical baseline alignment within row tracks. Cards sit axis-aligned by default. Rotation is permitted when it earns its place — e.g., a plate tilting up on hover, an artifact peeking out — but is never the default static layout move. Scattered placement and graph paper backgrounds remain retired.

---
> Source: [yankundesign/portfolio-2026](https://github.com/yankundesign/portfolio-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
