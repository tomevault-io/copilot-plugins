## therapist-services

> **Stillwater** is a single-page therapy services website exported from Figma using the Figma Make tool. It is a static, client-side React app with no backend. The site presents a fictional therapy practice with a service catalogue, therapist profile, testimonials, and a contact form.

# Therapy Services Website — CLAUDE.md

## Project Overview

**Stillwater** is a single-page therapy services website exported from Figma using the Figma Make tool. It is a static, client-side React app with no backend. The site presents a fictional therapy practice with a service catalogue, therapist profile, testimonials, and a contact form.

- **Stack:** React 18 · Vite 6 · TypeScript · Tailwind CSS v4 · shadcn/ui (Radix UI primitives)
- **Package manager:** npm (pnpm-workspace.yaml exists but README and runtime use npm)
- **Entry point:** `src/main.tsx` → mounts `<App />` into `#root`
- **Dev server:** `npm run dev` → `http://localhost:5173`
- **Build:** `npm run build`

---

## Directory Structure

```
therapy-figma-export/
├── index.html                      # HTML shell — sets title, meta description
├── package.json                    # Dependencies and scripts
├── vite.config.ts                  # Vite config — React plugin, Tailwind, path alias @/, figma asset resolver
├── postcss.config.mjs
├── pnpm-workspace.yaml
├── default_shadcn_theme.css        # Reference theme (not imported)
├── ATTRIBUTIONS.md
│
└── src/
    ├── main.tsx                    # App entry — createRoot → <App />
    │
    ├── app/
    │   ├── App.tsx                 # Entire page — all sections live here (single file, no routing)
    │   └── components/
    │       ├── figma/
    │       │   └── ImageWithFallback.tsx   # <img> wrapper with error fallback SVG
    │       └── ui/                         # shadcn/ui component library (42 files)
    │           ├── accordion.tsx
    │           ├── alert-dialog.tsx
    │           ├── alert.tsx
    │           ├── aspect-ratio.tsx
    │           ├── avatar.tsx
    │           ├── badge.tsx
    │           ├── breadcrumb.tsx
    │           ├── button.tsx
    │           ├── calendar.tsx
    │           ├── card.tsx
    │           ├── carousel.tsx
    │           ├── chart.tsx
    │           ├── checkbox.tsx
    │           ├── collapsible.tsx
    │           ├── command.tsx
    │           ├── context-menu.tsx
    │           ├── dialog.tsx
    │           ├── drawer.tsx
    │           ├── dropdown-menu.tsx
    │           ├── form.tsx
    │           ├── hover-card.tsx
    │           ├── input-otp.tsx
    │           ├── input.tsx
    │           ├── label.tsx
    │           ├── menubar.tsx
    │           ├── navigation-menu.tsx
    │           ├── pagination.tsx
    │           ├── popover.tsx
    │           ├── progress.tsx
    │           ├── radio-group.tsx
    │           ├── resizable.tsx
    │           ├── scroll-area.tsx
    │           ├── select.tsx
    │           ├── separator.tsx
    │           ├── sheet.tsx
    │           ├── sidebar.tsx
    │           ├── skeleton.tsx
    │           ├── slider.tsx
    │           ├── sonner.tsx
    │           ├── switch.tsx
    │           ├── table.tsx
    │           ├── tabs.tsx
    │           ├── textarea.tsx
    │           ├── toggle-group.tsx
    │           ├── toggle.tsx
    │           ├── tooltip.tsx
    │           ├── use-mobile.ts   # Hook: returns boolean via window.matchMedia (768px breakpoint)
    │           └── utils.ts        # cn() helper — merges clsx + tailwind-merge
    │
    └── styles/
        ├── index.css               # Imports fonts → tailwind → theme (in that order)
        ├── fonts.css               # Google Fonts import: Fraunces (serif) + Nunito (sans-serif)
        ├── tailwind.css            # @import 'tailwindcss'
        ├── theme.css               # CSS custom properties (:root + .dark) + @theme inline block
        └── globals.css             # Additional global styles (if any)
```

---

## App.tsx — Page Sections

The entire page is one component (`App.tsx`). No react-router; all navigation uses anchor links (`#services`, `#approach`, etc.).

### State

| Variable | Type | Purpose |
|---|---|---|
| `activeCategory` | `string` | Active filter tab in the Services section ("All" by default) |
| `menuOpen` | `boolean` | Controls mobile hamburger menu open/closed |
| `expandedService` | `number \| null` | ID of currently expanded service card (accordion toggle) |

### Data Constants (top of file)

| Constant | Type | Description |
|---|---|---|
| `services` | `array[12]` | Service objects: `id, category, title, description, duration, format, icon, color, accent, image` |
| `categories` | `string[]` | `["All", "Individual", "Couples & Family", "Group", "Specialized"]` — drives filter buttons |
| `testimonials` | `array[3]` | `{ quote, name, service }` — client quotes |

### Page Sections (in render order)

| Section | ID / Element | Description |
|---|---|---|
| **Navbar** | `<nav>` fixed top | Logo + nav links + CTA button. Mobile: hamburger toggles `menuOpen` |
| **Hero** | `<section>` | Headline, subtext, two CTA buttons, hero image |
| **Approach** | `id="approach"` | 3-pillar cards: "Whole-person care", "Evidence and intuition", "Unhurried pace" |
| **Services Catalogue** | `id="services"` | Filter tabs by category + responsive 3-col card grid. Each card has image, icon, title, description, duration/format badges, "Learn more" accordion toggle, "Book" CTA |
| **Therapist** | `id="therapists"` | Single therapist profile — Dr. Layla Osei, photo, specialties, bio, booking CTA |
| **Testimonials** | `<section>` | 3 client quote cards with 5-star ratings |
| **Contact / CTA** | `id="contact"` | Left: contact info (phone/email/address). Right: booking form (name, email, service select, message) — `e.preventDefault()`, no submission logic |
| **Footer** | `<footer>` | Logo, copyright, Privacy Policy / ToS / HIPAA links |

---

## Component: ImageWithFallback

**File:** `src/app/components/figma/ImageWithFallback.tsx`

Drop-in replacement for `<img>` that catches load errors and renders an inline SVG placeholder instead. Accepts all standard `React.ImgHTMLAttributes<HTMLImageElement>` props. Not used in the current `App.tsx` (which uses plain `<img>` tags directly), but available for future use.

---

## UI Components (`src/app/components/ui/`)

All 42 files are standard **shadcn/ui** components built on top of Radix UI primitives. They are pre-installed and available for use but **none are currently used** in `App.tsx` — the page uses Tailwind utility classes directly.

Key utilities in this folder:

- **`utils.ts`** — exports `cn(...inputs)` using `clsx` + `tailwind-merge`. Import as `import { cn } from "@/app/components/ui/utils"`.
- **`use-mobile.ts`** — exports `useIsMobile()` hook. Returns `true` when `window.innerWidth < 768px`.

---

## Styling & Design Tokens

### Fonts
- **Fraunces** (serif) — headings, blockquotes, logo. Applied inline via `style={{ fontFamily: "'Fraunces', serif" }}`
- **Nunito** (sans-serif) — body text. Applied on the root `<div>` in App.tsx

Both loaded from Google Fonts via `src/styles/fonts.css`.

### Color Palette (light mode, from `theme.css`)

| Token | Value | Usage |
|---|---|---|
| `--background` | `#F8F5F0` | Page background (warm off-white) |
| `--foreground` | `#1C2B1E` | Body text (dark forest green) |
| `--primary` | `#2D5A3D` | Buttons, navbar CTA, links (deep green) |
| `--primary-foreground` | `#F8F5F0` | Text on primary bg |
| `--secondary` | `#E8EDE5` | Section backgrounds, chips (soft sage) |
| `--accent` | `#C4714A` | Star ratings, service icons, labels (terracotta) |
| `--muted-foreground` | `#6B7A6E` | Subtext, nav links |
| `--card` | `#FFFFFF` | Service cards, testimonial cards |
| `--border` | `rgba(44,60,46,0.12)` | Subtle borders |

Dark mode tokens are defined under `.dark` but no toggle is wired up in the UI.

### Path Alias
`@` resolves to `src/` — configured in `vite.config.ts`.

---

## Key Dependencies

| Package | Version | Purpose |
|---|---|---|
| `react` / `react-dom` | 18.3.1 | UI framework (peer dep — installed via npm) |
| `vite` | 6.3.5 | Dev server and bundler |
| `@vitejs/plugin-react` | 4.7.0 | React Fast Refresh |
| `tailwindcss` + `@tailwindcss/vite` | 4.1.12 | Utility CSS |
| `lucide-react` | 0.487.0 | Icons used throughout App.tsx |
| `@radix-ui/*` | various | Primitives backing all shadcn/ui components |
| `class-variance-authority` | 0.7.1 | Variant styling in shadcn components |
| `tailwind-merge` + `clsx` | — | `cn()` helper |
| `react-router` | 7.13.0 | Installed but not used — no routing configured |

---

## Notes

- The contact form (`#contact`) has no submission handler — `onSubmit` calls `e.preventDefault()` only. Wire up to a backend or form service (e.g. Formspree, Resend) when needed.
- All images are Unsplash URLs. They require an internet connection and may not work offline.
- `pnpm-workspace.yaml` lists `supportedArchitectures: linux` only — use `npm` (not pnpm) on Windows/Mac to avoid arch errors.
- `react-router` v7 is installed as a dependency but zero routes are defined. If the project grows into multi-page, it is ready to use.
- shadcn/ui components in `src/app/components/ui/` are all available but unused — the current page is hand-crafted with raw Tailwind.

---
> Source: [Saatvikaggarwal/therapist_services](https://github.com/Saatvikaggarwal/therapist_services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
