## tech-book

> This document describes the design system, component patterns, and conventions used in this codebase to help integrate Figma designs accurately.

# CLAUDE.md — Figma Design Integration Guide

This document describes the design system, component patterns, and conventions used in this codebase to help integrate Figma designs accurately.

---

## Stack Overview

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router), React 19, TypeScript 5 |
| Styling | Tailwind CSS v3 + custom CSS variables |
| 3D / Canvas | Three.js, @react-three/fiber, @react-three/drei |
| Animation | GSAP, @react-spring/web, @use-gesture/react |
| Content | Markdown via gray-matter + remark/rehype pipeline |
| Deployment | Vercel (Analytics + Speed Insights) |
| Dev server | `npm run dev` — Next.js with Turbopack on port 5174 |
| Build | `npm run build` (runs `build:posts` then `next build`) |

---

## 1. Design Tokens

### CSS Variables (Source of Truth)

All color tokens are defined as CSS custom properties in [`src/styles/globals.css`](src/styles/globals.css).

**Light mode (`:root`):**
```css
--color-bg: #f4f7fb;
--color-bg-muted: #edf2f7;
--color-surface: #ffffff;
--color-surface-elevated: #f8fbfd;
--color-surface-muted: #e9eff6;
--color-surface-tint: #f2fbf8;
--color-text-primary: #0f172a;
--color-text-secondary: #475569;
--color-text-muted: #64748b;
--color-border: #d8e0ea;
--color-border-strong: #c4ceda;
--color-accent: #0f766e;         /* teal — primary interactive color */
--color-accent-hover: #115e59;
--color-accent-soft: rgba(15, 118, 110, 0.12);
--color-link: #0f766e;
--color-code: #eff4f9;
--shadow-soft: 0 18px 40px rgba(15, 23, 42, 0.06);
--shadow-medium: 0 24px 60px rgba(15, 23, 42, 0.08);
```

**Dark mode (`html.dark`):**
```css
--color-bg: #0b1320;
--color-surface: #111b2a;
--color-surface-elevated: #162234;
--color-text-primary: #e5edf6;
--color-text-secondary: #c5d0de;
--color-text-muted: #8fa2b7;
--color-accent: #58c9b9;         /* lighter teal in dark mode */
--color-accent-hover: #7edfd1;
--color-link: #7edfd1;
--color-code: #0f1727;
--shadow-soft: 0 18px 48px rgba(2, 8, 23, 0.28);
--shadow-medium: 0 28px 70px rgba(2, 8, 23, 0.34);
```

### Tailwind Token Extensions ([`tailwind.config.mjs`](tailwind.config.mjs))

```js
colors: {
  border: "var(--color-border)",   // shorthand Tailwind alias
  gray: "#868e96",
  dark: "#0F0F0F",
  bright: "#FFFFFF",
},
fontFamily: {
  press: ['"PressStart2P"', "cursive"],  // pixel/retro font for games UI
},
boxShadow: {
  "bg-glow": "inset 0 0 30px rgba(0,0,0,0.1)",
},
```

**Dark mode strategy:** `darkMode: "class"` — toggled by adding/removing `dark` class on `<html>`.

### How to reference tokens in new components

Always use CSS variables via the `[color:var(--color-*)]` Tailwind arbitrary-value syntax:

```tsx
// Correct
<p className="text-[color:var(--color-text-secondary)]">...</p>
<div className="border-[color:var(--color-border)]">...</div>

// Or via the 'border' Tailwind alias
<div className="border-border">...</div>
```

---

## 2. Typography

### Fonts

Defined in [`src/styles/fonts.css`](src/styles/fonts.css) as `@font-face` rules, served from `/public/fonts/`.

| Font | Weights | Usage |
|---|---|---|
| Pretendard | 400, 500, 600, 700 | All body/UI text (Korean + Latin) |
| PressStart2P | 400 | Games UI only (`font-press` Tailwind class) |

**Body font stack:**
```css
font-family: "Pretendard", "Apple SD Gothic Neo", "Noto Sans KR", "Malgun Gothic", sans-serif;
```

### Typography Scale Patterns

Used directly in prose content via `@tailwindcss/typography` plugin. The prose styles use CSS variables for colors so they respond to the theme automatically. Max prose width: `72ch`.

---

## 3. Utility Component Classes

Reusable semantic classes are defined in the `@layer components` block of [`src/styles/globals.css`](src/styles/globals.css). Always use these instead of recreating the patterns inline.

### Surfaces / Cards

```tsx
// Standard card — rounded-[28px], border, bg-surface, shadow-soft
<div className="surface-panel p-5">...</div>

// Elevated card — rounded-[32px], bg-surface-elevated, shadow-medium
<div className="surface-panel-strong p-6">...</div>

// Subtle inset — rounded-[12px], border, bg-surface-elevated
<div className="surface-subtle px-4 py-3">...</div>

// About/portfolio card (same as surface-panel)
<div className="about-card p-5">...</div>

// Interactive card with hover lift + color shift
<div className="about-card-interactive px-5 py-4">...</div>
```

### Text Utilities

```tsx
<p className="eyebrow-label">SECTION LABEL</p>         // uppercase, tracking-[0.24em], accent color
<h2 className="section-title">Title</h2>               // text-2xl sm:text-3xl, font-semibold
<p className="body-copy">Body text</p>                  // text-sm sm:text-base, leading-7, text-secondary
<p className="muted-copy">Smaller note</p>              // text-sm leading-6, text-muted
```

### Buttons

```tsx
<button className="button-primary">Primary CTA</button>    // filled, rounded-full, inverts on hover to accent
<button className="button-secondary">Secondary</button>    // outlined, rounded-full
```

### Interactive Elements

```tsx
<span className="tag-chip">Tag</span>        // pill chip, border, bg-surface-elevated
<input className="input-shell" />            // h-11, rounded-full, focus ring with accent-soft
<a className="nav-link" href="/">Link</a>   // text-secondary → text-primary on hover
<a className="accent-link" href="/">Link</a> // color-link → color-accent-hover on hover
```

### Layout Utilities

```tsx
<div className="page-shell">...</div>    // w-full py-8 sm:py-10
<div className="page-stack">...</div>    // flex flex-col gap-8 lg:gap-10
```

---

## 4. Project Structure

```
src/
├── app/                        # Next.js App Router
│   ├── layout.tsx              # Root layout (fonts, Header, Footer, ThemeInitializer)
│   ├── page.tsx                # Home / landing page
│   ├── (main)/                 # Route group — shared max-w-[1360px] container
│   │   ├── layout.tsx
│   │   ├── posts/
│   │   ├── games/
│   │   ├── categories/[category]/
│   │   ├── tags/[tag]/
│   │   └── search/
│   ├── about/
│   ├── projects/
│   └── ux-lab/
├── assets/
│   └── svg/                    # Inline SVG icon components
│       └── index.ts            # Barrel export with *Icon suffix
├── components/
│   ├── common/                 # Header, Footer, ThemeInitializer, GoogleAnalytics
│   ├── landing/                # Hero, section components for homepage
│   ├── layout/                 # Sidebar, PostsList, GameCardList, ClientHeaderWithSidebar
│   ├── posts/                  # PostListItem, PostContent, PostIndex, PostNavCard
│   ├── about/                  # AboutPage, ProjectCard, ProjectDetailView, ProjectVisual
│   ├── portfolio/              # PortfolioRail, SectionScrollNav
│   ├── projects/               # ProjectsPage, ProjectArchiveCard, ProjectPoster
│   └── games/                  # GameComponent, GameClientWrapper, guide/, ui/
├── games/                      # Self-contained canvas game engines (TS classes)
│   ├── pixel-runner/
│   └── space-shooter/
├── content/                    # Markdown source files
│   ├── posts/
│   └── games/
├── lib/                        # Data fetching, SEO helpers, formatters
├── styles/
│   ├── globals.css             # Tailwind directives + CSS variables + component classes
│   ├── fonts.css               # @font-face declarations
│   └── layout.css              # Layout-specific helpers (sticky-section)
└── types/                      # Shared TypeScript types (PostMeta, Post, Game)
```

**Path alias:** `@/` resolves to `src/` in all imports.

**Max content width:** `max-w-[1360px]` on the outer container; inner content often constrained further to `max-w-[980px]` or `max-w-[1040px]`.

---

## 5. Component Architecture

### Patterns

- **Server Components by default.** Client components are named with a `Client` suffix (e.g., `HeroPlaygroundClient.tsx`, `GameClientWrapper.tsx`, `UXLabClient.tsx`) and contain `"use client"` directive.
- **Props via typed interfaces** at the top of each file. No default prop pattern — use TypeScript optional fields.
- **No Storybook.** No component documentation system beyond the code itself.
- **Composition over configuration.** The `Header` accepts an `actions` slot (`React.ReactNode`) for injecting sidebar/mobile controls.

### Typical Component Pattern

```tsx
// Server component (default — no 'use client')
import { SomeType } from "@/types/something";

type Props = {
  items: SomeType[];
  variant?: "default" | "compact";
};

export default function MyComponent({ items, variant = "default" }: Props) {
  return (
    <section className="surface-panel p-5">
      <p className="eyebrow-label">Label</p>
      <h2 className="section-title mt-2">Title</h2>
      <div className="body-copy mt-4">...</div>
    </section>
  );
}
```

---

## 6. Icon System

Icons live in [`src/assets/svg/`](src/assets/svg/) as individual React components accepting `SVGProps<SVGSVGElement>`. They use `currentColor` for strokes/fills so they inherit the parent text color.

**Naming convention:** `PascalCase.tsx` for the file, exported with an `Icon` suffix via the barrel:

```ts
// src/assets/svg/index.ts
export { ChevronDown as ChevronDownIcon } from "@/assets/svg/ChevronDown";
export { Github as GithubIcon } from "@/assets/svg/Github";
// etc.
```

**Usage:**

```tsx
import { ChevronDownIcon } from "@/assets/svg";

<ChevronDownIcon className="h-4 w-4 text-[color:var(--color-accent)]" />
```

**Adding a new icon:** Create `src/assets/svg/IconName.tsx`, export the component, then add the re-export to `src/assets/svg/index.ts`.

---

## 7. Responsive Design

**Breakpoints:** Standard Tailwind (sm: 640px, lg: 1024px, xl: 1280px). The codebase primarily uses `sm:` and `lg:` breakpoints.

**Minimum viewport width:** `min-width: 350px` set on `body`.

**Common responsive patterns:**

```tsx
// Stack → row layout
<div className="flex flex-col gap-4 sm:flex-row sm:items-start">

// Single → multi-column grid
<div className="grid gap-3 sm:grid-cols-2">
<div className="grid gap-6 xl:grid-cols-2">
<div className="grid lg:grid-cols-[minmax(280px,360px)_minmax(0,1fr)]">

// Hidden on mobile, visible on desktop
<nav className="hidden items-center gap-5 lg:flex">
```

**Container width:** Outer shell always `mx-auto max-w-[1360px] px-4 sm:px-6 lg:px-8`.

---

## 8. Asset Management

### Images
- Stored in `public/images/`
- Always use Next.js `<Image>` component with explicit `width`/`height` or `fill` + `sizes` prop
- `sizes` prop is always set for responsive images

```tsx
<Image
  src="/images/Profile.JPG"
  alt="Description"
  fill
  className="object-cover"
  sizes="(min-width: 1200px) 220px, (min-width: 768px) 30vw, 44vw"
/>
```

### Fonts
- Stored in `public/fonts/` as `.woff2` files
- Loaded via `@font-face` in [`src/styles/fonts.css`](src/styles/fonts.css) with `font-display: swap`

### 3D Models / HDR
- `.glb` models in `public/models/`
- `.hdr` environment maps in `public/hdr/`

### Game Assets
- Sprite sheets and PNGs in `public/assets/pixel-runner/` and `public/assets/space-shooter/`

---

## 9. Styling Approach

### Method
**Tailwind CSS utility classes** are the primary styling method. There are no CSS Modules, Styled Components, or other CSS-in-JS solutions.

### Rules
1. Use the token-aware `[color:var(--color-*)]` Tailwind syntax for all colors that must respond to dark mode.
2. Use the semantic component classes (`.surface-panel`, `.eyebrow-label`, etc.) before writing custom Tailwind combos.
3. Transitions use `duration-200` or `duration-300` with `ease-out`. Hover lifts are always `-translate-y-0.5` or `-translate-y-1`.
4. Border radius conventions: cards `rounded-[28px]` / `rounded-[32px]`, subtle elements `rounded-[12px]`, pills `rounded-full`.
5. Do not write raw hex colors in className strings — map them to the CSS variable system.

### Global Styles Entry Point
[`src/styles/globals.css`](src/styles/globals.css) is imported at the root layout. It contains:
- `@import` of `layout.css`, `fonts.css`, and `prismjs/themes/prism-tomorrow.css`
- `@tailwind base / components / utilities`
- All CSS variable token definitions
- All `@layer components` utility classes
- Keyframe animations

---

## 10. Theme System

Theme is toggled by adding/removing the `dark` class on `<html>`. The `ThemeInitializer` component injects a blocking script to read `localStorage` and apply the class before first paint (prevents flash).

Initial state on server render: `<html style={{ colorScheme: "light" }}>` — light mode by default.

---

## 11. Figma-to-Code Mapping

When implementing a Figma design, follow this mapping:

| Figma layer type | Code pattern |
|---|---|
| Card / surface | `.surface-panel`, `.surface-panel-strong`, `.about-card` |
| Section label / eyebrow | `.eyebrow-label` |
| Section heading | `.section-title` or inline `text-2xl font-semibold` |
| Body paragraph | `.body-copy` |
| Small/muted text | `.muted-copy` |
| Primary CTA button | `.button-primary` |
| Secondary/outlined button | `.button-secondary` |
| Tag / pill chip | `.tag-chip` |
| Text input | `.input-shell` |
| Nav link | `.nav-link` |
| Accent/teal text | `text-[color:var(--color-accent)]` |
| Primary text | `text-[color:var(--color-text-primary)]` |
| Secondary text | `text-[color:var(--color-text-secondary)]` |
| Muted text | `text-[color:var(--color-text-muted)]` |
| Border | `border-[color:var(--color-border)]` |
| Page background | `bg-[color:var(--color-bg)]` |
| Card background | `bg-[color:var(--color-surface)]` |
| Elevated surface | `bg-[color:var(--color-surface-elevated)]` |
| Subtle/tinted surface | `bg-[color:var(--color-surface-tint)]` |
| Soft shadow | `shadow-[var(--shadow-soft)]` (or via `.surface-panel`) |
| Medium shadow | `shadow-[var(--shadow-medium)]` |
| Accent soft background | `bg-[color:var(--color-accent-soft)]` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seonghoho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
