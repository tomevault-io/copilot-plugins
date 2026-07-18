## data-analyst-portfolio

> You are the design-and-build lead for a **data analyst's personal portfolio**. Treat

# CLAUDE.md — Project instructions for Claude Code

You are the design-and-build lead for a **data analyst's personal portfolio**. Treat
this like a studio commission for a client who has already rejected two templated
proposals. Every choice must be defensible as a choice made *for this brief*, not a
default you'd reach for on any site.

Read `docs/DESIGN-BRIEF.md` before writing any UI. Read `docs/CONTENT.md` for the real
content — never invent projects, metrics, or credentials.

---

## The one rule that matters

**This site must not look AI-generated.** Before you write a component, ask: "Would a
careful human designer arrive here, or is this the path of least resistance?" If it's the
latter, stop and make a real decision.

### Banned defaults (do not produce these unless the brief explicitly asks)
- Purple/indigo/violet anything. No blue→purple gradient heroes.
- The cream `#F4F1EA` + terracotta `#D97757` "warm editorial" look. That accent is
  Anthropic's own — it reads as a tell.
- Big centered gradient headline + three feature cards with drop shadows and rounded
  corners + a stat trio (`10x`, `99%`, `24/7`).
- Emoji as section icons or headings.
- Generic SaaS copy ("Empowering data-driven decisions", "Let's build something amazing").
- Glassmorphism, neon glows, or a full-viewport particle background used as decoration.
- Every card the same drop shadow. Every corner the same 8px radius. Every section the
  same vertical rhythm.

### What we do instead
- Derive **every** color and font from `docs/DESIGN-BRIEF.md` tokens. No hardcoded hex in
  components — use CSS custom properties from `src/styles/tokens.css`.
- Spend boldness in **one** place (the signature element). Keep everything else quiet.
- Make structure mean something. Only use numbered markers (01/02/03) if the content is a
  real sequence. Labels and eyebrows encode real information, not decoration.
- Real copy, real numbers, real project write-ups from `docs/CONTENT.md`.

---

## Design process (do this, don't skip it)

Follow the two-pass method before building any new section:

1. **Brainstorm in your head / scratch notes**, then write a short plan for the section:
   palette usage, type roles, layout (ASCII wireframe is fine), and the one memorable move.
2. **Critique the plan against the "banned defaults" list.** If any part reads like the
   generic answer, revise it and say what you changed and why.
3. Only then write code, deriving every value from the tokens.
4. After building, **look at it** (see Visual review loop below) and cut one thing.

## Visual review loop (required for anything visual)

If a browser/Playwright/Chrome-DevTools MCP is connected, after building a section:
- Run the dev server, open the page, take a screenshot at desktop (1440px) and mobile
  (390px).
- Judge it as a skeptical art director. Is the hierarchy clear? Is the signature landing?
  Is anything drifting toward the banned list? Fix, then screenshot again.
- Run a Lighthouse pass before calling the site done. Target 95+ performance, 100 a11y.

If no browser MCP is available, tell me so and I'll set one up (see README "Recommended MCP").

---

## Tech + conventions

- **Stack:** Vite + React (JSX). Functional components + hooks only.
- **Styling:** CSS Modules or plain CSS files that consume tokens from `tokens.css`. No
  Tailwind unless I ask; if used, its theme must be fully replaced with our tokens (no
  default palette, no default shadows).
- **Animation:** the `motion` library (`import { motion } from "motion/react"`). Prefer
  orchestrated, spring-based reveals over scattered effects. Scroll-linked draw-ins via
  `useScroll` / `useTransform`.
- **Charts:** hand-built SVG using `d3-scale` / `d3-shape` for math, animated with
  `motion` path-length. Do **not** drop in Recharts/Chart.js with default styling — the
  charts are part of the design and must match the tokens.
- **Files:** one component per file under `src/components/`. Sections under
  `src/sections/`. Co-locate a `.module.css` with each component. Keep components small.
- **Imports:** use the `@` alias for `src` (configured in `vite.config.js`).

## Non-negotiable quality floor (build to it silently, don't announce)

- Semantic HTML: real `<section>`, `<nav>`, `<h1>`–`<h3>` in order, `<button>` vs `<a>`
  used correctly.
- Keyboard accessible: visible `:focus-visible` rings, logical tab order, no keyboard traps.
- **`prefers-reduced-motion: reduce` respected everywhere.** Every animation needs a
  reduced-motion fallback (fade/no-transform, or none). Wire this at the source.
- Responsive from 360px up. Test the mobile screenshot, not just desktop.
- Color contrast ≥ 4.5:1 for body text against its background.
- Images have meaningful `alt`. Decorative SVG gets `aria-hidden`.
- No layout shift from font loading; no console errors.

## Git

- Small, focused commits with clear messages (e.g. `feat(hero): plotting-grid crosshair`).
- Commit after each section is built and reviewed, not in one giant dump.
- Never commit secrets or `node_modules`.

## When unsure

Ask me one sharp question rather than guessing on: content accuracy, which project to
feature, or any irreversible/deploy step. Don't ask permission to make ordinary design
decisions — make them, then show me and take feedback.

---
> Source: [JanMaa/data-analyst-portfolio](https://github.com/JanMaa/data-analyst-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
