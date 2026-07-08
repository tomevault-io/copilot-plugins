## baby-tracker

> This file is the source of truth for how Claude Code should approach all web projects in this workspace. It covers design system usage, coding conventions, and Supabase backend rules. Read it fully before beginning any task.

# CLAUDE.md — Web Development Baseline

This file is the source of truth for how Claude Code should approach all web projects in this workspace. It covers design system usage, coding conventions, and Supabase backend rules. Read it fully before beginning any task.

---

## 1. DESIGN SYSTEM

All projects reference the award-winning design system documented below. Do not invent ad-hoc colours, spacing, or typography — use the variables and patterns defined here.

### 1.1 How to Apply the Design System

- **Existing projects:** Check whether a palette has already been established (look for `:root` CSS variables). If one exists, extend it using the master template variables. Do not replace an established palette unless explicitly instructed.
- **New projects:** Ask which palette to use before writing any styles. Present the six options from section 1.3 with a one-line description each.
- **Always:** Implement using CSS custom properties (`--color-primary`, etc.) — never hardcode hex values into component styles.

---

### 1.2 CSS Custom Properties Master Template

Paste into the root stylesheet and override palette variables per project:

```css
/* ============================================
   DESIGN SYSTEM — Root Variables
   ============================================ */

:root {
  /* — PALETTE (swap these per project) — */
  --color-bg:         #F5F0E8;
  --color-surface:    #FFFFFF;
  --color-primary:    #49C5B6;
  --color-secondary:  #DF6C4F;
  --color-accent:     #ECD06F;
  --color-text:       #111111;
  --color-text-muted: #6B6B6B;
  --color-border:     rgba(0,0,0,0.1);

  /* — TYPOGRAPHY — */
  --font-display: 'DM Sans', sans-serif;
  --font-body:    'DM Sans', sans-serif;

  --text-xs:   0.75rem;
  --text-sm:   0.875rem;
  --text-base: 1rem;
  --text-lg:   1.25rem;
  --text-xl:   1.75rem;
  --text-2xl:  2.5rem;
  --text-3xl:  clamp(2.5rem, 5vw, 4rem);
  --text-hero: clamp(4rem, 12vw, 10rem);

  --weight-regular: 400;
  --weight-bold:    700;
  --weight-black:   900;

  --leading-tight:  0.9;
  --leading-snug:   1.2;
  --leading-normal: 1.5;
  --leading-loose:  1.75;

  --tracking-tight:  -0.04em;
  --tracking-normal:  0;
  --tracking-wide:    0.08em;

  /* — SPACING — */
  --space-xs:  0.5rem;
  --space-sm:  1rem;
  --space-md:  2rem;
  --space-lg:  4rem;
  --space-xl:  8rem;
  --space-2xl: 14rem;

  /* — SHAPE — */
  --radius-sm:   4px;
  --radius-md:   12px;
  --radius-lg:   24px;
  --radius-full: 9999px;

  /* — ANIMATION — */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-out:   cubic-bezier(0.4, 0, 0.2, 1);
  --duration-fast: 0.2s;
  --duration-base: 0.4s;
  --duration-slow: 0.7s;

  /* — SHADOWS — */
  --shadow-sm: 0 2px 8px rgba(0,0,0,0.06);
  --shadow-md: 0 8px 24px rgba(0,0,0,0.1);
  --shadow-lg: 0 20px 60px rgba(0,0,0,0.15);
}
```

---

### 1.3 Colour Palettes

Six palettes are available. Each is pre-built as a `:root` override block — paste over the palette section of the master template.

---

#### Palette A — "Terracotta Studio"
**Mood:** Bold, minimal, warm. Best for food & drink, lifestyle, portfolios.

```css
:root {
  --color-primary:    #DF6C4F;
  --color-bg:         #F5F0EB;
  --color-surface:    #FFFFFF;
  --color-text:       #111111;
  --color-accent:     #1A1A1A;
}
```

**Key trait:** Single saturated terracotta accent on warm off-white. Use sparingly for CTAs only.

---

#### Palette B — "Teal & Bloom"
**Mood:** Fresh, playful, modern. Best for e-commerce, wellness, app landing pages.

```css
:root {
  --color-primary:    #49C5B6;
  --color-secondary:  #FF9398;
  --color-accent:     #ECD06F;
  --color-bg:         #FAFAFA;
  --color-text:       #1C1C2E;
  --color-surface:    #FFFFFF;
}
```

**Key trait:** Scroll-triggered background colour transitions — each section owns one of the three accent colours.

---

#### Palette C — "Electric Neon"
**Mood:** High-energy, tech, disruptive. Best for startups, dev tools, fintech, Web3.

```css
:root {
  --color-primary:    #C8FF00;
  --color-bg:         #0D1230;
  --color-surface:    #161A3A;
  --color-text:       #FFFFFF;
  --color-accent:     #3D6EFF;
  --color-highlight:  #FF4D3D;
}
```

**Key trait:** Dark-first. Electric lime dominates the navbar/logo. CTAs in bright blue stand out strongly against the dark field.

---

#### Palette D — "Carnival Burst"
**Mood:** Vibrant, cultural, editorial. Best for music, events, entertainment, media.

```css
:root {
  --color-red:        #D4421A;
  --color-yellow:     #F5E642;
  --color-green:      #38C98C;
  --color-purple:     #9B88CC;
  --color-bg:         #111111;
  --color-text:       #111111;
  --color-text-inv:   #FFFFFF;
}
```

**Key trait:** Each section uses a different palette colour as a full-bleed background. Hover effects swap tile colours. Bold condensed typography fills each block.

---

#### Palette E — "Editorial Cream"
**Mood:** Sophisticated, luxury, French minimalism. Best for agencies, luxury brands, editorial, law, architecture.

```css
:root {
  --color-primary:    #1B1F8A;
  --color-bg:         #F5F0E8;
  --color-surface:    #EDE8DC;
  --color-text:       #1A1A1A;
  --color-accent:     #C0392B;
  --color-secondary:  #2C4A3E;
}
```

**Key trait:** Background transitions on scroll. Typography is the hero — large display serifs plus tight condensed sans-serifs. Minimal imagery.

---

#### Palette F — "Illustrated Vivid"
**Mood:** Creative, joyful, expressive. Best for illustrators, children's brands, education, cultural projects.

```css
:root {
  --color-primary:    #8B45CC;
  --color-secondary:  #F5E642;
  --color-accent:     #E87D30;
  --color-green:      #3DAF5A;
  --color-bg:         #FFF8F0;
  --color-text:       #111111;
}
```

**Key trait:** Full-bleed illustrated hero. Nav as overlay triggered by circular button. Hover on nav items changes background colour.

---

### 1.4 Quick Reference — Palette by Site Type

| Site Type | Palette | Typography Style |
|---|---|---|
| Tech / SaaS / Startup | C — Electric Neon | Expressive Display |
| E-commerce / Product | B — Teal & Bloom | Expressive Display |
| Creative Agency / Portfolio | F or E | Either |
| Food / Lifestyle / Brand | A — Terracotta Studio | Expressive Display |
| Luxury / Editorial / Professional | E — Editorial Cream | Editorial Serif |
| Music / Events / Culture | D — Carnival Burst | Expressive Display |

---

### 1.5 Typography Systems

Use one system consistently per project — do not mix.

#### Style 1 — "Expressive Display"
Condensed black sans-serif hero, clean geometric body. The contrast in size is the design.

```css
--font-display: 'Bebas Neue', 'Anton', sans-serif;
--font-body:    'DM Sans', 'Inter', 'Manrope', sans-serif;

--text-hero:    clamp(5rem, 14vw, 12rem);
--text-display: clamp(3rem, 8vw, 7rem);
--text-heading: clamp(1.5rem, 4vw, 3rem);
--text-body:    1rem;
--text-small:   0.875rem;

--leading-tight:  0.88;
--leading-normal: 1.5;
--tracking-tight: -0.04em;
--tracking-wide:  0.08em;
```

#### Style 2 — "Editorial Serif"
High-contrast serif display, clean sans-serif body. Elegant and editorial.

```css
--font-display: 'DM Serif Display', 'Playfair Display', serif;
--font-body:    'DM Sans', 'Inter', sans-serif;
--font-label:   'Space Grotesk', 'Barlow Condensed', sans-serif;

--text-hero:    clamp(4rem, 10vw, 9rem);
--text-display: clamp(2.5rem, 5vw, 5rem);
--text-heading: 1.75rem;
--text-body:    1rem;

--leading-display: 1.05;
--leading-body:    1.65;
--tracking-display: -0.02em;
--tracking-label:   0.12em;
```

---

### 1.6 Layout Principles

- **Whitespace is a design element.** A section with nothing but a bold headline on a coloured background is a valid, powerful composition.
- **Full-bleed over contained.** Extend images, colours, and type edge-to-edge. Use `width: 100vw; margin-inline: calc(50% - 50vw);` to break out of constrained wrappers.
- **Asymmetric grids.** Prefer `grid-template-columns: 65fr 35fr` or `7fr 3fr` over equal columns.
- **Sticky minimal navbar.** Keep height under 60px. Transparent, gaining `backdrop-filter: blur(12px)` on scroll.
- **CTAs as colour anchors.** Primary buttons always use the strongest accent as a solid fill, with inverse text colour. Secondary actions are outline-only. Use pill shape (`border-radius: 9999px`).

---

### 1.7 UI Patterns

Apply these patterns from the design system — do not reinvent them.

**Scroll-triggered colour transitions** — CSS `animation-timeline: view()` or GSAP ScrollTrigger to shift `background-color` between sections.

**Full-bleed section cards** — `min-height: 100vh`, solid colour background, hero headline at `clamp(4rem, 12vw, 10rem)`.

**Colour-coded sections/products** — Each item gets a `data-color` attribute; CSS `var(--card-bg)` swaps on hover with `transition: background-color 0.4s var(--ease-out-expo)`.

**Hover menu overlays** — Full-screen overlay slides in (`translateY(-100% → 0)`). Menu items trigger background colour changes in the overlay.

**Dark-first design** — For Palette C projects, design dark by default. Lime accent reserved for interactive elements only.

**Custom cursor** — Hide OS cursor, replace with a small `position: fixed` dot that scales on hover over interactive elements.

---

### 1.8 Animation Principles

- **Purposeful only.** Every animation guides attention, confirms an action, or communicates state. No decorative animations.
- **Preferred easing:** `cubic-bezier(0.16, 1, 0.3, 1)` — fast start, soft end.
- **Entrance animations:** `opacity: 0→1`, `translateY: 30px→0`, 0.6s, staggered 0.1s for siblings.
- **Hover microinteractions:** Scale `1→1.02`, shadow increase, inner image `1→1.06`. Duration 0.3s.
- **Text reveal on load:** Split into words/chars, animate with `translateY(110%→0)` behind `overflow: hidden`.

---

## 2. CODING CONVENTIONS

### 2.1 General

- JavaScript style is **project-dependent** — check the existing codebase before writing any JS. If starting fresh, ask which approach to use (vanilla, React, or vanilla + libraries).
- Always use **semicolons**.
- Prefer `const` and `let` over `var`.
- Use `async/await` over `.then()` chains.
- Add comments on non-obvious logic; do not comment self-evident code.
- Keep functions small and single-purpose.

### 2.2 HTML

- Semantic elements first — use `<nav>`, `<main>`, `<section>`, `<article>`, `<header>`, `<footer>` correctly.
- All interactive elements must be keyboard accessible.
- Images must have meaningful `alt` attributes.

### 2.3 CSS

- All colours, spacing, radii, shadows, and durations must use CSS custom properties — never hardcoded values in component styles.
- Mobile-first media queries (`min-width`).
- Use `clamp()` for fluid typography and spacing.
- Avoid `!important` unless overriding a third-party library with no alternative.

### 2.4 File Organisation

- One concern per file — do not mix unrelated functionality in the same script or stylesheet.
- Shared utilities go in a clearly named file (`utils.js`, `helpers.js`) — do not duplicate logic across files.
- CSS variables always live in `:root` in the main/root stylesheet, never scoped to a component unless intentionally overriding.

---

## 3. SUPABASE RULES

These rules apply to every project using Supabase. Never deviate from them.

### 3.1 Client Initialisation

The Supabase client **must always be named `sb`**. This is non-negotiable — the UMD bundle registers `window.supabase`, and naming the client anything else will cause breakage in static HTML projects.

```js
const sb = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

Do not name it `client`, `supabaseClient`, `supabase`, or anything else.

### 3.2 Row Level Security

- **Always assume RLS is enabled.** Never write queries that assume unrestricted access.
- Never suggest disabling RLS to fix a query problem — fix the policy instead.
- When writing new queries, check whether the intended operation is covered by existing RLS policies before assuming it will work.

### 3.3 Authentication

- Use Supabase Auth for all authentication — do not implement custom auth systems.
- Always handle the `onAuthStateChange` listener to reactively update UI on session changes.
- Store nothing sensitive in `localStorage` beyond what Supabase Auth manages itself.

### 3.4 Project References

- Each project has its own Supabase project URL and anon key. **Do not assume these from memory or from another project.**
- If the project ref is not in the current file or provided in the task, ask before proceeding.

### 3.5 Queries

- Prefer the Supabase JS client (REST API) over raw SQL in the browser context.
- Always handle errors explicitly — do not silently swallow `{ data, error }` responses:

```js
const { data, error } = await sb.from('table').select('*');
if (error) {
  console.error('Query failed:', error.message);
  return;
}
```

- For realtime features, use Supabase Realtime Broadcast for signaling/ephemeral data and database listeners for persistent state — not the other way around.

---

## 4. BEFORE YOU START ANY TASK

1. **Read this file fully** if you haven't in this session.
2. **Check the existing codebase** — note the palette in use, the JS style, and any existing Supabase setup before writing anything.
3. **Ask first if unclear** — especially about palette choice for new projects, or Supabase project credentials.
4. **Do not over-engineer.** These are static HTML/JS projects with a Supabase backend. Keep solutions simple and direct.

---
> Source: [pw-tf/baby-tracker](https://github.com/pw-tf/baby-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
