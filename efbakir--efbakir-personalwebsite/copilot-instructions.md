## efbakir-personalwebsite

> Vanilla static site (HTML + CSS + JS, no framework, no build step). Deployed on Vercel.

# Claude Code — efbakir.com

## Project Overview

Vanilla static site (HTML + CSS + JS, no framework, no build step). Deployed on Vercel.

- `index.html` — projects homepage (three-column sticky layout)
- `writings/index.html` — writings page (three-column sticky layout)
- `glossary/index.html` — glossary page (three-column sticky layout)
- `projects/*/index.html` — individual project pages
- `design-system.css` — all design tokens (CSS custom properties)
- `styles.css` — all component and layout styles (imports design-system.css)
- `site.js` — shared behaviour: theme toggle, CV modal, project filters, writings nav, glossary accordion
- `nav.js` — mobile navigation and scroll lock

---

## Figma MCP Integration

**Figma file key:** `S9CygczNMjBBtiBcz0mOzU`

### Required flow for every Figma-driven implementation

1. Run `get_design_context` on the target node to get structured code + layout
2. Run `get_screenshot` for visual reference
3. Translate the React + Tailwind output to **vanilla HTML + CSS** following this project's conventions
4. Map all Tailwind colors → `var(--*)` tokens from `design-system.css`
5. Map all Tailwind spacing → `var(--spacing-*)` tokens
6. Check `styles.css` for an existing class before writing a new one
7. Validate visually against the Figma screenshot before finishing

### IMPORTANT rules

- **Never use Tailwind, React, or any framework** — this is plain HTML/CSS/JS
- **Never hardcode hex colors** — always use semantic tokens: `var(--color-text-primary)`, `var(--accent)`, etc.
- **Never hardcode pixel spacing** — always use `var(--spacing-N)` tokens
- **Never install new packages** — the only dependency is `@vercel/analytics`
- **Never add inline styles** — all styles belong in `styles.css`
- **Do not add Figma capture scripts** to HTML unless explicitly asked for a Figma capture task

---

## Design Tokens

All tokens live in `design-system.css`. Never define values outside this file.

### Colors

```css
/* Brand */
--accent: #FC5113;               /* Orange — hover, focus, active states */
--color-accent-soft: rgba(252, 81, 19, 0.18); /* Soft accent background */

/* Core palette */
--color-primary: #141414;
--color-gray-lg: #8C8C8C;
--color-gray-md: #B4B4B4;
--color-gray-sm: #f1f1f1;

/* Semantic — USE THESE in components, never the raw palette above */
--color-bg                  /* page background */
--color-surface             /* card / panel surface */
--color-surface-muted       /* slightly dimmer surface */
--color-border              /* all borders */
--color-text-primary        /* headings, primary text */
--color-text-secondary      /* body text, descriptions */
--color-text-tertiary       /* labels, metadata, placeholders */
--color-text-inverse        /* text on dark backgrounds */
--color-overlay             /* modal backdrop */
--color-overlay-strong      /* strong backdrop */
```

**Dark mode:** All semantic tokens automatically remap under `[data-theme="dark"]`. Never write separate dark-mode color values in components — use only semantic tokens.

### Typography

Two typefaces only:

| Token prefix | Family | Usage |
|---|---|---|
| `--body-*` / `--label-*` | `"Suisse Int'l", sans-serif` | All prose, UI labels |
| `--label-heading-*` | `"Geist Mono", monospace` | Mono labels, nav items, metadata tags |

**Text style tokens:**

```css
/* Body */
--body-1-*   18px / 28px / 0px tracking       /* large body, bio */
--body-2-*   16px / 24px / -0.08px tracking   /* article body */
--body-3-*   14px / 20px / -0.035px tracking  /* small body, nav titles */

/* Labels */
--label-2-*         15px / 22px / 0px
--label-3-*         14px / 20px / 0px
--label-heading-*   12px / 16px / -0.1px  Geist Mono  /* section labels, UPPERCASE */
--label-heading-sm  13px / 18px / -0.1px  Geist Mono

/* Display (headings) */
--display-lg-font-size: 24px  / --display-lg-line-height: 32px
--display-md-font-size: 22px  / --display-md-line-height: 1.35
--display-sm-font-size: 20px
```

### Spacing (4px base grid)

```css
--spacing-0:  0
--spacing-1:  4px
--spacing-2:  8px
--spacing-3:  12px
--spacing-4:  16px
--spacing-5:  20px
--spacing-6:  24px
--spacing-8:  32px
--spacing-10: 40px
--spacing-12: 48px
--spacing-14: 56px
--spacing-16: 64px
--spacing-20: 80px
--spacing-24: 96px
```

### Motion

```css
--duration-fast: 0.2s
--duration-base: 0.3s
--easing-standard: ease
--transition-color     /* color + background-color + border-color */
--transition-opacity
--transition-transform
```

### Radius

```css
--radius-sm: 2px
--radius-md: 4px
--radius-lg: 8px
--radius-pill: 999px
```

### Shadows

```css
--shadow-modal
--shadow-modal-dark
--shadow-soft
--shadow-soft-dark
```

### Border shorthand

```css
--border: 1px solid var(--color-border)
```

---

## Layout System

All pages use a **three-column CSS Grid** sticky layout:

- **LEFT** — always sticky (bio/header panel)
- **MIDDLE** — scrollable list
- **RIGHT** — sticky article/detail panel

Key layout tokens:

```css
--grid-container-width: 1280px
--page-padding-top: var(--spacing-16)   /* 64px */
--page-padding-bottom: var(--spacing-8) /* 32px */
--section-gap: var(--spacing-8)
--section-gap-large: var(--spacing-12)
--size-panel-sticky-top: 96px           /* offset for sticky panels */
```

---

## Component Patterns

### CSS class naming

BEM-inspired, kebab-case, descriptive:
- `.writing-article-nav` → block
- `.writing-article-nav-item` → element
- `.writing-article-nav-next` → modifier

### Interactive elements

All hover/focus transitions use shared tokens:

```css
/* Hover color change */
transition: var(--transition-color);

/* Focus ring — applied globally to all :focus-visible */
outline: var(--focus-ring-width) solid var(--focus-ring-color);
outline-offset: var(--focus-ring-offset);
```

Hover state for text links uses `--accent` for color.

### Buttons and nav items

Use `<button>` for in-page actions (no page navigation), `<a>` for links. Both share the same visual treatment via shared CSS selectors.

### Hidden state

Use the HTML `hidden` attribute, not `display: none` in CSS. The CSS handles `[hidden] { display: none !important; }` where needed.

### Tags / mono labels

Uppercase monospace labels (nav items, section labels, metadata) use `--label-heading-font-family` (Geist Mono) with `letter-spacing: var(--label-heading-letter-spacing)`.

---

## JavaScript Patterns

- All JS is vanilla ES5-compatible, no modules, no bundler
- Behaviour is initialized inside a `DOMContentLoaded` listener in `site.js`
- Feature functions follow the pattern: `function initFeatureName() { ... }`
- Interactive list → panel switching (writings, glossary) uses a shared `activate(id, skipTransition)` pattern
- Dark mode uses `document.documentElement.setAttribute('data-theme', 'dark'|'light')` + localStorage
- `hidden` attribute toggled directly for show/hide, not CSS classes

---

## Asset Handling

- SVG project thumbnails: `assets/project-thumbs/*.svg`
- WebP assets: `assets/project-thumbs/*.webp`
- Figma MCP image assets: use the provided `https://www.figma.com/api/mcp/asset/...` URLs directly as `src`
- Site logo: `star.svg` at root
- **Do not install icon libraries** — use inline SVG or existing assets only

---

## File Editing Rules

- **`design-system.css`** — tokens only. No component styles.
- **`styles.css`** — all component and layout styles. Organized by component section with `/* === */` dividers.
- **`site.js`** — shared behaviour. Add new `initFeature()` functions; call them inside `DOMContentLoaded`.
- **`nav.js`** — mobile nav only. Do not add unrelated behaviour here.
- **HTML files** — semantic HTML5. Each page is self-contained with `<header>`, `<main>`, and optional `<footer>`.

---
> Source: [efbakir/efbakir-personalwebsite](https://github.com/efbakir/efbakir-personalwebsite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
