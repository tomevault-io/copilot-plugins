## textql-landing-page-new-test

> > "Good design is as little design as possible."

# TextQL Landing Page - Development Guide

## Dieter Rams Design System

> "Good design is as little design as possible."
> — Dieter Rams

This landing page follows Dieter Rams' 10 Principles of Good Design, inspired by his work at Braun. The theme prioritizes function over decoration, clarity over complexity.

### Design Principles

1. **Innovative** - Fresh approach to data analytics presentation
2. **Useful** - Every element serves a purpose
3. **Aesthetic** - Pure, honest beauty through restraint
4. **Understandable** - Self-explanatory interface
5. **Unobtrusive** - Design serves content, not ego
6. **Honest** - No deceptive patterns or gratuitous effects
7. **Long-lasting** - Timeless, not trendy
8. **Thorough** - Every detail considered
9. **Environmentally friendly** - Minimal visual pollution
10. **As little design as possible** - Less, but better

---

## Color Palette

Inspired by Braun product design. Colors have meaning.

### Neutrals (Primary)

| Token | Hex | Usage |
|-------|-----|-------|
| `white` | #FFFFFF | Pure backgrounds |
| `off-white` | #FAFAF8 | Warm section backgrounds |
| `gray-50` - `gray-900` | Scale | Text hierarchy, borders |
| `black` | #0D0D0C | Maximum contrast |

### Functional Colors (Use Sparingly)

| Token | Hex | Usage |
|-------|-----|-------|
| `orange` | #E86A33 | Primary actions (Braun signature) |
| `orange-hover` | #D45A25 | Hover state |
| `green` | #2E7D32 | Success states only |
| `red` | #C62828 | Error states only |
| `blue` | #1565C0 | Links and information only |

---

## Section Backgrounds

Two options. Simple.

```tsx
<Section background="light" />  // off-white (cream) with paper dots texture
<Section background="dark" />   // forest-900 with paper dots texture
```

**IMPORTANT**: All sections must use the `Section` component to get the paper dots texture automatically. This includes:
- Dot grid pattern
- Fade splotches for depth variation
- Canvas-generated grayscale noise

Disable texture with `noTexture`:

```tsx
<Section background="light" noTexture />
<Section background="light" bordered />  // adds subtle border
<Section background="light" padding="none" className="min-h-[85vh]" />  // custom sizing
```

The texture is implemented via `PaperDotsBackground` component internally - never use CSS classes for textures.

---

## Typography Scale

Consistent sizing through theme tokens. Never use hardcoded clamp/rem/px values for headings.

| Class | Usage | Size Range |
|-------|-------|------------|
| `text-display` | Hero headlines only | 2rem → 3rem |
| `text-heading-2` | Section headers | 1.75rem → 2.5rem |
| `text-heading-3` | Subsection headers | 1.5rem → 2rem |
| `text-stat` | Large data values | 2.5rem → 3.5rem |

```tsx
// Bad - hardcoded values
<h1 className="text-[clamp(2.5rem,5vw,4.5rem)]">
<h2 className="text-[clamp(2rem,4vw,3rem)] leading-[1.1] tracking-[-0.02em]">

// Good - theme tokens
<h1 className="text-display font-semibold">
<h2 className="text-heading-2 font-semibold">
```

---

## Section Padding

All sections must use consistent horizontal padding:

```tsx
// Standard section padding
px-8 md:px-16 lg:px-24  // 32px → 64px → 96px
```

The `Container` component applies this automatically. Direct section content should match.

---

## Theme Audit

One command to verify design compliance:

```bash
npm run audit
```

Checks for:
- **Colors** - Hardcoded hex, rgb, non-theme Tailwind colors
- **Fonts** - Hardcoded font-family (should use `var(--font-sans)`)
- **Typography** - Hardcoded clamp/rem font sizes (should use `text-display`, `text-heading-2`, etc.)
- **Border Radius** - Hardcoded px values
- **Sections** - Raw `<section>` tags (should use `<Section>` component)
- **Page Structure** - Missing `<Navbar />` or `<Footer />` in page.tsx files
- **Page Layout** - Hardcoded max-width/width/margin on page wrappers

### Fixing Violations

```tsx
// Bad
<div className="bg-[#f59e0b]">
<h2 className="text-[clamp(2.5rem,5vw,4.5rem)]">
<section className="py-24">
<main className="max-w-[1800px]">

// Good
<div className="bg-orange">
<h2 className="text-heading-2 font-semibold">
<Section background="light" padding="xl">
<main className="min-h-screen">
```

---

## Animation Philosophy

> "Indifference towards people and the reality in which they live is actually the one and only cardinal sin in design."

Animation should aid understanding, not decorate. We animate only when:

- It provides feedback on user action
- It guides attention purposefully
- It aids spatial understanding

We do NOT animate:
- Decorative flourishes (floating particles, pulsing glows)
- Gratuitous hover effects (scale, bounce)
- Attention-seeking loops

Respect `prefers-reduced-motion` always.

---

## Legacy Compatibility

Old tokens are mapped to new colors for gradual migration:

| Old Token | Maps To |
|-----------|---------|
| `cream-*` | `gray-*` / `off-white` |
| `forest-*` | `gray-*` |
| `accent-emerald` | `green` |
| `accent-gold` | `orange` |
| `accent-rust` | `orange` |
| `accent-cyan` | `blue` |

---

## Before Committing

```bash
npm run audit   # Check theme compliance
npm run build   # Verify build
```

Less, but better.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheEthanDing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
