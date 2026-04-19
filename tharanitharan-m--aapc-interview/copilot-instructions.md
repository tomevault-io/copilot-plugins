## aapc-interview

> AAPC brand design system — colors, typography, spacing and UI conventions


# AAPC Design System

AAPC is the world's largest healthcare education and credentialing organization.
The brand is **corporate-professional with a warm, accessible edge** — authoritative without being intimidating.
Teal is the single dominant accent color. Never introduce competing accent colors.

---

## Color Palette

Use CSS custom properties defined in `globals.css`. Never hardcode hex values inline.

| Role | CSS Variable | Hex | Usage |
|---|---|---|---|
| **Primary Teal** | `var(--color-primary)` | `#0D8484` | Logo, CTAs, primary buttons, links, nav highlights, active states |
| **Light Teal** | `var(--color-primary-light)` | `#E8F5F2` | Background tints, hover states, sidebar active background |
| **Dark Text** | `var(--color-text-dark)` | `#1A1A1A` | Headings, primary text |
| **Body Gray** | `var(--color-text-body)` | `#555555` | Body copy, descriptions |
| **Muted Gray** | `var(--color-text-muted)` | `#888888` | Placeholder text, captions, disabled states |
| **Background** | `var(--color-bg)` | `#FFFFFF` | Page background |
| **Surface** | `var(--color-surface)` | `#FFFFFF` | Cards, modals, dropdowns |
| **Border** | `var(--color-border)` | `#E0E0E0` | Input borders, card borders, dividers |
| **Error** | `var(--color-error)` | `#DC2626` | Error states, destructive actions |

### In Tailwind (use arbitrary values when needed)

```tsx
// Primary teal
className="bg-[#0D8484] text-white"
className="text-[#0D8484]"
className="border-[#0D8484]"

// Light teal tint
className="bg-[#E8F5F2]"

// Text
className="text-[#1A1A1A]"   // headings
className="text-[#555555]"   // body
className="text-[#888888]"   // muted
```

---

## Typography

### Font Setup
- **Hero headings**: Merriweather (serif) — bold/black weight, warm and institutional
- **All other text**: Lato (sans-serif) — clean, highly legible

```tsx
// Hero headline — serif, large, bold
className="font-serif text-5xl font-black text-[#1A1A1A] leading-tight"

// Section headings
className="font-sans text-2xl font-bold text-[#1A1A1A]"

// Sub-headings
className="font-sans text-lg font-semibold text-[#1A1A1A]"

// Body text
className="font-sans text-base font-normal text-[#555555] leading-relaxed"

// Small / caption
className="font-sans text-sm text-[#888888]"

// Teal link
className="text-[#0D8484] font-semibold hover:underline transition-colors"
```

---

## Spacing & Layout

The design uses an 8px base grid. Sections breathe — use generous vertical margins.

```tsx
// Page container (max 1280px, centered)
className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8"

// Section vertical spacing
className="py-12 md:py-16 lg:py-20"

// Space between major elements within a section
className="space-y-8" or className="gap-8"

// Three-column service card grid
className="grid grid-cols-1 gap-8 md:grid-cols-3"

// Two-column layout (sidebar + content)
className="grid grid-cols-1 gap-8 lg:grid-cols-[280px_1fr]"
```

### Card
```tsx
className="rounded-lg border border-[#E0E0E0] bg-white p-6 shadow-sm"
```

### Section dividers
```tsx
className="border-t border-[#E0E0E0] my-12"
```

---

## UI Components

### Primary CTA Button ("Join Us Today →")
Pill-shaped, solid teal. This is the most important button style.
```tsx
className="rounded-full bg-[#0D8484] px-8 py-3 text-white font-semibold text-base transition-colors hover:bg-[#0A6F6F] inline-flex items-center gap-2"
```

### Secondary CTA (text link with arrow)
```tsx
className="text-[#0D8484] font-semibold hover:underline transition-colors inline-flex items-center gap-1"
// Example: <span>Get Started <span aria-hidden="true">→</span></span>
```

### Outline Button (e.g., "Sign Up" in nav)
```tsx
className="rounded-full border border-[#0D8484] text-[#0D8484] px-6 py-2 font-semibold text-sm transition-colors hover:bg-[#E8F5F2]"
```

### Input / Select
```tsx
className="w-full rounded-md border border-[#E0E0E0] bg-white px-3 py-2.5 text-[#1A1A1A] placeholder:text-[#888888] focus:outline-none focus:ring-2 focus:ring-[#0D8484] transition-shadow"
```

### Sidebar Navigation (active state)
```tsx
// Active item
className="border-l-4 border-[#0D8484] bg-[#E8F5F2] pl-4 text-[#0D8484] font-semibold"
// Inactive item
className="pl-4 text-[#555555] hover:text-[#0D8484] hover:bg-[#E8F5F2] transition-colors"
```

### Info Banner
```tsx
className="w-full bg-slate-100 border-y border-[#E0E0E0] py-3 px-4 text-sm text-[#555555] flex items-center gap-2"
```

---

## General UI Rules

- All interactive elements must have `transition-colors` or `transition-all`
- Buttons are **pill-shaped** (`rounded-full`) for primary actions, `rounded-md` for secondary
- Always add `aria-label` to icon-only buttons
- Use semantic HTML: `<main>`, `<section>`, `<article>`, `<nav>`, `<header>`, `<footer>`
- Whitespace is generous — never crowd elements. When in doubt, add more padding.
- The audience may skew older or less tech-native — prioritize legibility over density
- Line height for body text: always `leading-relaxed` (1.625) or `leading-loose` (2)
- WCAG AA compliance: always verify teal text on white passes contrast (large text: ✅, small text: verify size ≥ 18.66px bold or 24px regular)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tharanitharan-M) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
