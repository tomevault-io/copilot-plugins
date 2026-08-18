## mobile-responsive

> Mobile-first responsive design conventions for Recurv. Apply to every component, page section, and layout file.


# Mobile-First Responsive Architecture

## The Golden Rule — inline style is for tokens only

**Inline `style` props are ONLY for design tokens:** colors, font-family, letter-spacing, border-color, and specific brand values from `theme.ts`.

**All structural properties MUST use Tailwind classes** so they can carry responsive prefixes (`sm:`, `md:`, `lg:`):
- `display`, `grid`, `flex`
- `gridTemplateColumns`, `gridColumn`
- `padding`, `margin`, `gap`
- `width`, `height`, `maxWidth`
- Font sizes for headings (use `var(--fs-*)` CSS custom properties instead)

```tsx
// ❌ Wrong — inline style for layout
<div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", padding: "96px 56px" }}>

// ✅ Correct — Tailwind for layout, inline for tokens only
<div className="grid grid-cols-1 lg:grid-cols-2 py-16 lg:py-24 px-4 lg:px-14"
     style={{ borderTop: `1px solid ${t.line}` }}>
```

## Layout Primitives — always use these

Every section must be wrapped in the `<Container>` component. Do not hardcode gutters.
`<Container>` adds responsive horizontal padding only — it does NOT apply a max-width or center the content block.

```tsx
import { Container } from "@/components/Container";
import { Section } from "@/components/Section";

// Basic usage
<Section>
  <Container>
    {/* content */}
  </Container>
</Section>

// With custom background / border
<Section className="bg-[#0F0E14]" style={{ borderTop: `1px solid ${t.line}` }}>
  <Container>
    {/* content */}
  </Container>
</Section>
```

## Breakpoints

Use Tailwind defaults — do not invent custom ones:

| Prefix | Width     | Use case                   |
|--------|-----------|----------------------------|
| (base) | 0–639px   | Mobile portrait — design here first |
| `sm:`  | 640px     | Mobile landscape / large phone |
| `md:`  | 768px     | Tablet                     |
| `lg:`  | 1024px    | Desktop                    |
| `xl:`  | 1280px    | Wide desktop (design target) |

## Grid collapse convention

Every multi-column grid must start single-column on mobile and expand up:

```tsx
// ❌ Wrong
<div style={{ display: "grid", gridTemplateColumns: "1.05fr 1fr", gap: 56 }}>

// ✅ Correct
<div className="grid grid-cols-1 gap-8 lg:grid-cols-[1.05fr_1fr] lg:gap-14">
```

Common patterns:
- 2-col → `grid-cols-1 md:grid-cols-2`
- 3-col → `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`
- 5-col features → `grid-cols-1 sm:grid-cols-2 lg:grid-cols-5`
- Sidebar layouts → `grid-cols-1 lg:grid-cols-[1fr_1.4fr]`

## Typography

Never use hardcoded pixel font sizes for headings. Use the fluid scale from `globals.css`:

```tsx
// ❌ Wrong
<h1 style={{ fontSize: 78 }}>

// ✅ Correct
<h1 style={{ fontSize: "var(--fs-hero)", fontFamily: t.fontDisplay, fontWeight: 500 }}>
```

Available scale tokens:
- `--fs-hero` — hero h1 (clamp ~40px → 78px)
- `--fs-display` — oversized display (footer wordmark, contact hero)
- `--fs-h2-xl` — large section headings (clamp ~32px → 56px)
- `--fs-h2-lg` — medium section headings (clamp ~28px → 48px)
- `--fs-h2-md` — standard headings (clamp ~24px → 44px)

## Section component files

Each distinct page section lives in its own file for reusability:
- Home page sections → `components/home/`
- Contact page sections → `components/contact/`
- Shared across pages → `components/sections/`

```
components/
  home/
    HeroSection.tsx
    RecurvCoreSection.tsx
    CollectionTypesSection.tsx
    DashboardSection.tsx
    IndustriesSection.tsx
    BlogSection.tsx
    SecuritySection.tsx
    StatsSection.tsx
    CtaSection.tsx
  contact/
    ContactHeroSection.tsx
    ContactDoorsSection.tsx
    ContactFormSection.tsx
```

## Mobile navigation

`SiteHeader` tracks `menuOpen: boolean`. Below `md:` the nav links and CTA are hidden; a hamburger icon is shown. Never add nav items without accounting for the mobile state.

## Checklist before committing a component

- [ ] No structural/layout values in inline `style` (only token colors/fonts)
- [ ] Uses `<Container>` for horizontal padding
- [ ] Multi-column grids have a `grid-cols-1` base class
- [ ] Heading font sizes use `var(--fs-*)` CSS custom properties
- [ ] Tested visually at 390px (iPhone 14), 768px (iPad), and 1280px (desktop)

---
> Source: [ClickMod/RecurvwebV2](https://github.com/ClickMod/RecurvwebV2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
