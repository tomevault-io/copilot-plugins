## website-cloner

> <!-- AUTO-GENERATED from AGENTS.md — do not edit directly.

<!-- AUTO-GENERATED from AGENTS.md — do not edit directly.
     Run `bash scripts/sync-agent-rules.sh` to regenerate. -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Website Reverse-Engineer Template

## What This Is
A reusable template for reverse-engineering any website into a clean, modern Next.js codebase using AI coding agents. The Next.js + shadcn/ui + Tailwind v4 base is pre-scaffolded — just run `/clone-website <url1> [<url2> ...]`.

## Tech Stack
- **Framework:** Next.js 16 (App Router, React 19, TypeScript strict)
- **UI:** shadcn/ui (Radix primitives, Tailwind CSS v4, `cn()` utility)
- **Animation:** GSAP 3 (all plugins free) via `@gsap/react` `useGSAP` — the default for scroll-driven, sequenced, split-text, morph, and physics-based motion
- **Icons:** Lucide React (default — will be replaced/supplemented by extracted SVGs)
- **Styling:** Tailwind CSS v4 with oklch design tokens
- **Deployment:** Vercel

## Commands
- `npm run dev` — Start dev server
- `npm run build` — Production build
- `npm run lint` — ESLint check
- `npm run typecheck` — TypeScript check
- `npm run check` — Run lint + typecheck + build

## Code Style
- TypeScript strict mode, no `any`
- Named exports, PascalCase components, camelCase utils
- Tailwind utility classes, no inline styles
- 2-space indentation
- Responsive: mobile-first

## Design Principles
- **Pixel-perfect emulation** — match the target's spacing, colors, typography exactly
- **No personal aesthetic changes during emulation phase** — match 1:1 first, customize later
- **Real content** — use actual text and assets from the target site, not placeholders
- **Beauty-first** — every pixel matters
- **Motion-perfect emulation** — animations are part of the design. Match the target's easing, duration, stagger, and scroll behavior, not just the end state.

## Animation (GSAP)
Reproduce a target's motion with **GSAP**, not hand-rolled CSS, whenever the behavior is scroll-linked, pinned, scrubbed, sequenced (timelines), split-text, morphing, parallax, or physics/inertia-based. Plain CSS transitions are acceptable only for trivial hover/focus color changes.

Conventions (see `docs/GSAP_GUIDE.md` for full patterns — read it before building any animated section):
- Import GSAP and every plugin from **`@/lib/gsap`** (never bare `"gsap"`) — plugins are pre-registered there and SSR-safe.
- Any component that animates begins with `"use client"`.
- Drive **every** animation through the **`useGSAP()`** hook with `{ scope: ref }` — it auto-reverts on unmount and is Strict-Mode safe. Never manually add/remove tweens in a bare `useEffect`.
- For inertia/smooth scrolling (target uses Lenis, Locomotive, or eased scroll), mount **`<SmoothScroller>`** — the GSAP-native (ScrollSmoother) replacement — instead of adding those libraries. Mount it only when the target actually smooth-scrolls.
- Match exact values extracted from the target: ease name (or `CustomEase` for bespoke curves), duration, delay, stagger, and ScrollTrigger `start`/`end`/`scrub`/`pin`. Do not guess.
- Wrap responsive or reduced-motion variants in `gsap.matchMedia()`.

## Project Structure
```
src/
  app/              # Next.js routes
  components/       # React components
    ui/             # shadcn/ui primitives
    icons.tsx       # Extracted SVG icons as React components
    SmoothScroller.tsx # GSAP ScrollSmoother wrapper (mount only for smooth-scroll sites)
  lib/
    utils.ts        # cn() utility (shadcn)
    gsap.ts         # Central GSAP plugin registration + re-exports (import GSAP from here)
  types/            # TypeScript interfaces
  hooks/            # Custom React hooks
public/
  images/           # Downloaded images from target site
  videos/           # Downloaded videos from target site
  seo/              # Favicons, OG images, webmanifest
docs/
  research/         # Inspection output (design tokens, components, layout)
  design-references/ # Screenshots and visual references
scripts/            # Asset download scripts
```

## MOST IMPORTANT NOTES
- When launching Claude Code agent teams, ALWAYS have each teammate work in their own worktree branch and merge everyone's work at the end, resolving any merge conflicts smartly since you are basically serving the orchestrator role and have full context to our goals, work given, work achieved, and desired outcomes.
- After editing `AGENTS.md`, run `bash scripts/sync-agent-rules.sh` to regenerate platform-specific instruction files.
- After editing `.claude/skills/clone-website/SKILL.md`, run `node scripts/sync-skills.mjs` to regenerate the skill for all platforms.

# Website Inspection Guide

## How to Reverse-Engineer Any Website

This guide outlines what to capture when inspecting a target website via Chrome MCP or browser DevTools.

## Phase 1: Visual Audit

### Screenshots to Capture
- [ ] Every distinct page — desktop, tablet, mobile
- [ ] Dark mode variants (if applicable)
- [ ] Light mode variants (if applicable)
- [ ] Key interaction states (hover, active, open menus, modals)
- [ ] Loading/skeleton states
- [ ] Empty states
- [ ] Error states

### Design Tokens to Extract
- [ ] **Colors** — background, text (primary/secondary/muted), accent, border, hover, error, success, warning
- [ ] **Typography** — font family, sizes (h1-h6, body, caption, label), weights, line heights, letter spacing
- [ ] **Spacing** — padding/margin patterns (look for a scale: 4px, 8px, 12px, 16px, 24px, 32px, etc.)
- [ ] **Border radius** — buttons, cards, avatars, inputs
- [ ] **Shadows/elevation** — card shadows, dropdown shadows, modal overlay
- [ ] **Breakpoints** — when does the layout shift? (inspect with DevTools responsive mode)
- [ ] **Icons** — which icon library? custom SVGs? sizes?
- [ ] **Avatars** — sizes, shapes, fallback behavior
- [ ] **Buttons** — all variants (primary, secondary, ghost, icon-only, danger)
- [ ] **Inputs** — text fields, textareas, selects, checkboxes, toggles

## Phase 2: Component Inventory

For each distinct UI component, document:
1. **Name** — what would you call this component?
2. **Structure** — what HTML elements / child components does it contain?
3. **Variants** — does it have different sizes, colors, or states?
4. **States** — default, hover, active, disabled, loading, error, empty
5. **Responsive behavior** — how does it change at different breakpoints?
6. **Interactions** — click, hover, focus, keyboard navigation
7. **Animations** — transitions, entrance/exit animations, micro-interactions

### Common Components to Look For
- Navigation (top bar, sidebar, bottom bar)
- Cards / list items
- Buttons and links
- Forms and inputs
- Modals and dialogs
- Dropdowns and menus
- Tabs and segmented controls
- Avatars and user badges
- Loading skeletons
- Toast notifications
- Tooltips and popovers

## Phase 3: Layout Architecture

- [ ] **Grid system** — CSS Grid? Flexbox? Fixed widths?
- [ ] **Column layout** — how many columns at each breakpoint?
- [ ] **Max-width** — main content area max-width
- [ ] **Sticky elements** — header, sidebar, floating buttons
- [ ] **Z-index layers** — navigation, modals, tooltips, overlays
- [ ] **Scroll behavior** — infinite scroll, pagination, virtual scrolling

## Phase 4: Technical Stack Analysis

- [ ] **Framework** — React? Vue? Angular? Check `__NEXT_DATA__`, `__NUXT__`, `ng-version`
- [ ] **CSS approach** — Tailwind (utility classes), CSS Modules, Styled Components, Emotion, vanilla CSS
- [ ] **State management** — Redux (check DevTools), React Query, Zustand, Pinia
- [ ] **API patterns** — REST, GraphQL (check network tab for `/graphql` requests)
- [ ] **Font loading** — Google Fonts, self-hosted, system fonts
- [ ] **Image strategy** — CDN, lazy loading, srcset, WebP/AVIF
- [ ] **Animation library** — GSAP (check `window.gsap`, `ScrollTrigger`, `ScrollSmoother`), Framer Motion, or CSS transitions only. **This template is GSAP-driven** — GSAP + all plugins are preinstalled and reproduce the widest range of motion.
- [ ] **Smooth scroll** — Lenis (`.lenis`), Locomotive (`[data-scroll-container]`), or GSAP ScrollSmoother (`#smooth-wrapper`). Reproduce with the scaffolded `<SmoothScroller>`.
- [ ] **If GSAP:** dump `window.ScrollTrigger.getAll()` per section and record real `start`/`end`/`scrub`/`pin`/`trigger` values; capture each tween's ease, duration, delay, and stagger. See `docs/GSAP_GUIDE.md`.

## Phase 5: Documentation Output

After inspection, create these files in `docs/research/`:
1. `DESIGN_TOKENS.md` — All extracted colors, typography, spacing
2. `COMPONENT_INVENTORY.md` — Every component with structure notes
3. `LAYOUT_ARCHITECTURE.md` — Page layouts, grid system, responsive behavior
4. `INTERACTION_PATTERNS.md` — Animations, transitions, hover states, and GSAP specifics (per-section ScrollTrigger start/end/scrub/pin, tween ease/duration/stagger, timeline step order)
5. `TECH_STACK_ANALYSIS.md` — What the site uses and our chosen equivalents

---
> Source: [Rijal-Madan-777/website-cloner](https://github.com/Rijal-Madan-777/website-cloner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
