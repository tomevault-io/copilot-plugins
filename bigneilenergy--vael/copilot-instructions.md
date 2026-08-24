## vael

> vael is a web-based design system generator built for freelance designers and SaaS product teams. It takes brand parameters as input and extrapolates a complete, production-ready design token system using the Anthropic Claude API.

# vael — CLAUDE.md

## What is vael?

vael is a web-based design system generator built for freelance designers and SaaS product teams. It takes brand parameters as input and extrapolates a complete, production-ready design token system using the Anthropic Claude API.

The long-term goal is to make vael a shareable tool for the broader design community — a BYOK (bring your own key) web app.

---

## Critical rules — read before every task

### Build verification (mandatory)
After every task that modifies existing files, run:
```
npm run build
```
If the build fails, show the exact error and fix it before marking the task complete. Do not report a task as done if the build has errors. A passing build prevents white screen crashes in the browser.

### Auto-compact threshold
Context window auto-compacts at 30% remaining. This is intentional — do not fight it.

### Scope discipline
Only modify files explicitly mentioned in the prompt. Do not touch unrelated files even if you notice issues. If you see a problem outside the scope, note it but do not fix it without being asked.

### Commit reminders
Always remind Neil to commit to git before starting a task and after completing it.

---

## Core interaction model

**Two-panel layout:**
- Left panel: live component preview canvas
- Right panel: parameter controls + Generate button

**Two modes:**

1. **Live preview (no API call)** — the canvas updates in real time as the user adjusts parameters. Color pickers, radius chips, density toggles, dark mode switch all update the canvas instantly. This is a direct translation of raw inputs into rendered components — no Claude involvement.

2. **Generate (API call)** — clicking Generate sends the parameters to the Claude API. Claude extrapolates the full token system: complete color scales (50–950), semantic colors, spacing ramp, shadow scale, motion tokens, and component-level tokens. The canvas updates to reflect the generated system. Export becomes available after generation.

**Post-generate tweaks** apply locally on top of the generated system without a new API call. The user must click Generate again to fully recalculate.

---

## Parameter set

| Section | Parameters |
|---|---|
| Brand | name, primary color, accent color, tone |
| Typography | display font, body font |
| Geometry | border radius scale, spacing density |
| Motion | motion style, easing (enter/exit/hover), speed multiplier, interaction preset |
| Interactions | preset, trigger, target |
| Output | dark mode toggle, export format |

**Tone options:** professional, playful, editorial, minimal, bold, enterprise

**Radius options:** sharp, subtle, moderate, rounded, pill

**Density options:** compact, comfortable, spacious

**Motion options:** none, subtle, expressive

**Interaction presets:** border trace, fill sweep, shimmer, pulse ring, magnetic

**Export formats:** tailwind.config.js, CSS variables, Tokens Studio JSON

---

## Token architecture

### Pre-generate (live preview)
Before Generate is clicked, the canvas is driven directly by raw param values:
- Primary color → used as-is for buttons, focus rings, accents
- Accent color → used as-is for highlights
- Dark mode → toggles between dark/light surface values
- Radius scale → maps to a fixed lookup table of pixel values
- Density → maps to a fixed spacing multiplier

### Post-generate (full system)
After Generate, the canvas is driven by the Claude-generated token object:

```js
{
  colors: {
    primary:  { 50, 100, 200, 300, 400, 500, 600, 700, 800, 900 },
    accent:   { 50, 100, 200, 300, 400, 500, 600, 700, 800, 900 },
    neutral:  { 0, 50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950, 1000 },
    semantic: { success, warning, error, info }
  },
  typography: {
    displayFont, bodyFont,
    scale:         { xs, sm, base, lg, xl, 2xl, 3xl, 4xl, 5xl },
    weight:        { light, regular, medium, semibold, bold },
    lineHeight:    { tight, snug, normal, relaxed },
    letterSpacing: { tight, normal, wide, wider }
  },
  spacing: { px, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24 },
  radius:  { none, xs, sm, md, lg, xl, 2xl, full },
  shadow:  { xs, sm, md, lg, xl },
  motion: {
    duration: { instant, fast, normal, slow, slower },
    easing:   { default, enter, exit, spring }
  },
  components: {
    button: { paddingX, paddingY, fontSize, fontWeight, radius, primaryBg, primaryText, primaryHover },
    input:  { paddingX, paddingY, fontSize, radius, borderColor, bg, focusBorder },
    card:   { padding, radius, bg, borderColor, shadow },
    badge:  { paddingX, paddingY, fontSize, radius, primaryBg, primaryText }
  }
}
```

---

## Token naming convention

All generated tokens follow this structured naming schema:

### Primitives — `{category}-{scale}`
- `color-primary-500`
- `font-size-lg`
- `spacing-4`
- `radius-md`

### Semantics — `{role}-{variant}-{state}`
- `surface-default`
- `text-primary`
- `border-focus`
- `feedback-success`

### Component tokens — `{component}-{element}-{property}-{state}`
- `button-primary-bg-default`
- `button-primary-bg-hover`
- `input-border-focus`
- `card-surface-bg-default`

### Rules
- All lowercase, hyphen-separated
- No camelCase, no underscores
- All three export formats must reflect this naming schema

---

## Component library scope

35 components across 7 categories, all in src/components/preview/components/:

**Interactive:** PreviewButton, PreviewInput, PreviewTextarea, PreviewCheckbox, PreviewRadio, PreviewToggle, PreviewSlider, PreviewSelect

**Navigation:** PreviewNav, PreviewBreadcrumb, PreviewTabs, PreviewPagination, PreviewSidebar

**Feedback:** PreviewAlert, PreviewToast, PreviewBadge, PreviewProgress, PreviewSkeleton, PreviewSpinner

**Layout:** PreviewCard, PreviewStatCards, PreviewModal, PreviewAvatar

**Data:** PreviewTable (TanStack v8), PreviewTooltip

**Overlays:** PreviewCommand, PreviewDropdownMenu, PreviewContextMenu, PreviewPopover, PreviewSheet

**Patterns:** PreviewForm, PreviewCombobox, PreviewCalendar, PreviewCollapsible, PreviewResizable

---

## Tech stack

- **Framework:** Vite + React (no TypeScript)
- **Styling:** Tailwind CSS + inline styles for token-driven values
- **Components:** shadcn/ui
- **Icons:** Lucide React
- **API:** Anthropic Claude API, direct browser fetch with `anthropic-dangerous-direct-browser-access: true`
- **Model:** claude-opus-4-5
- **Fonts:** Google Fonts, dynamically loaded per session
- **Auth:** BYOK — user provides Anthropic API key, never stored server-side
- **Table:** TanStack Table v8

---

## Design language (pure zinc palette)

- **Shell background:** #0a0a0a
- **Canvas background:** #0d0d0d
- **Panel background:** #0a0a0a
- **Borders:** #1c1c1c
- **Input background:** #161616
- **Input border:** #252525
- **Active chip background:** #282828
- **Active chip color:** #cccccc
- **Active chip border:** #3a3a3a
- **Inactive chip color:** #666666
- **Section labels:** #aaaaaa (NOT darker — must be legible)
- **Primary text:** #e8e8e8
- **Secondary text:** #888888
- **Muted/helper text:** #777777 minimum
- **Generate button:** white background, black text
- **Font:** DM Mono for all UI chrome

---

## Coding conventions

- Use Tailwind utility classes for layout and spacing in the shell
- Use inline styles for any value that comes from the generated token system
- Keep preview components self-contained — each receives `system` and `params` as props
- Avoid useEffect for derived state — compute from props inline
- Component files should be small and focused — split early
- Export logic lives in `src/lib/export.js`
- Claude prompt lives in `src/lib/prompt.js`
- Pre-generate token maps live in `src/lib/tokens.js`
- shadcn primitives in `src/components/ui/` — do not modify these directly

---

## Pre-generate token maps (tokens.js)

```js
export const radiusMap = {
  sharp:    { none:"0px", xs:"0px", sm:"0px", md:"2px",  lg:"4px",  xl:"6px",  full:"9999px" },
  subtle:   { none:"0px", xs:"1px", sm:"2px", md:"4px",  lg:"6px",  xl:"8px",  full:"9999px" },
  moderate: { none:"0px", xs:"2px", sm:"4px", md:"6px",  lg:"8px",  xl:"12px", full:"9999px" },
  rounded:  { none:"0px", xs:"4px", sm:"6px", md:"10px", lg:"14px", xl:"20px", full:"9999px" },
  pill:     { none:"0px", xs:"6px", sm:"10px",md:"16px", lg:"24px", xl:"32px", full:"9999px" },
}

export const densityMap = {
  compact:     { xs:"2px",  sm:"4px",  md:"8px",  lg:"12px", xl:"16px" },
  comfortable: { xs:"4px",  sm:"8px",  md:"12px", lg:"16px", xl:"24px" },
  spacious:    { xs:"6px",  sm:"12px", md:"16px", lg:"24px", xl:"32px" },
}
```

---

## Export formats

| Format | Output | Use case |
|---|---|---|
| tailwind.config.js | JS config object | Tailwind projects |
| CSS variables | :root { --token: value } | Any web project |
| Tokens Studio | W3C-style JSON | Figma via Tokens Studio plugin |

---
> Source: [bigneilenergy/vael](https://github.com/bigneilenergy/vael) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
