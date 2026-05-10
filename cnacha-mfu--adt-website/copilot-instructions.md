## adt-website

> This is the replacement website for **adt.mfu.ac.th** (School of Applied Digital Technology, Mae Fah Luang University). The live site at https://adt.mfu.ac.th/ is the reference for content accuracy; this project supersedes it with a modern interactive design.

# ADT Website — Claude Code Instructions

This is the replacement website for **adt.mfu.ac.th** (School of Applied Digital Technology, Mae Fah Luang University). The live site at https://adt.mfu.ac.th/ is the reference for content accuracy; this project supersedes it with a modern interactive design.

## Dev server

```
npm run dev        # http://localhost:3000
start.bat          # double-click shortcut on Windows
```

## Stack

| Layer | Tech |
|---|---|
| Framework | Next.js 16, App Router, TypeScript |
| Styling | Tailwind CSS 3 with custom design tokens |
| Animation | Framer Motion 11, Canvas API, IntersectionObserver |
| State / persistence | React Context + localStorage (`adt-website-data`) |
| Icons | React Icons 5 |
| Date utils | date-fns 3 |
| IDs | uuid v4 |

## Design system — "Digital Frontier"

Dark navy base with teal and gold accents. Never deviate from these tokens.

**Colors (tailwind.config.ts)**
- `void` `#030B1A` — page background
- `deep` `#070F20` — section backgrounds
- `teal` `#0CC8D4` — primary accent, interactive elements
- `gold` `#F5A623` — secondary accent, highlights
- `ink-primary` `#EEF2FF` — headings / primary text
- `ink-secondary` `#94A3B8` — body text
- `border` `#1A2D4A` — card borders

**Fonts**
- `font-bebas` — display / hero numbers
- `font-syne` — section headings (Prompt auto-used for Thai glyphs)
- `font-dm` — body copy (Prompt auto-used for Thai glyphs)
- `font-prompt` — Thai-primary text; also the fallback for all Thai glyphs in Syne/DM Sans

**Utility classes (globals.css)**
- `.glass-card` — frosted card with backdrop blur
- `.gradient-border` — subtle teal border gradient on hover
- `.teal-glow` — text shadow glow
- `.shimmer-text` — animated gradient text

## Project structure

```
src/
  app/
    page.tsx                  # Home (Hero → Carousel → News → Programs → Staff → Footer)
    news/page.tsx             # Filterable news listing
    staff/page.tsx            # Filterable staff directory
    admin/
      layout.tsx              # Auth wrapper + sidebar (password: admin123 via sessionStorage)
      page.tsx                # Dashboard with stats
      staff/page.tsx          # Staff CRUD
      highlights/page.tsx     # Highlights CRUD
      news/page.tsx           # News CRUD
  components/
    Navbar.tsx                # Fixed header, EN/TH toggle, mobile menu
    Hero.tsx                  # Particle canvas + animated counters + parallax
    AnimatedGrid.tsx          # Canvas dot-grid background
    HighlightCarousel.tsx     # Auto-rotating highlight banners
    NewsSection.tsx           # Featured + grid news display
    ProgramsSection.tsx       # 6 degree programs (static)
    StaffSection.tsx          # Staff grid preview (top 6 active)
    ScrollReveal.tsx          # IntersectionObserver animation wrapper
    Footer.tsx
  context/
    AppContext.tsx             # All data state + CRUD + language toggle
  lib/
    types.ts                  # StaffMember, Highlight, NewsItem, AppData interfaces
    initial-data.ts           # Seed data (6 staff, 4 highlights, 5 news items)
  hooks/
    useScrollReveal.ts        # useScrollReveal + useCounterAnimation hooks
public/
  logo.png                    # ADT logo (white, transparent bg) — used in Navbar
asset/                        # Source assets (not served — copy to public/ to use)
```

## Data model

All data lives in `src/lib/types.ts`. Key relationships:

- **StaffMember** — `role: 'dean' | 'assoc_dean' | 'faculty' | 'researcher' | 'staff'`, bilingual fields (`name`/`nameTH`, `title`/`titleTH`, etc.), `order: number`, `active: boolean`
- **Highlight** — `type: 'event' | 'celebration' | 'achievement' | 'announcement'`, optional `ctaUrl`, `startDate`/`endDate`, `active: boolean`
- **NewsItem** — `category: 'news' | 'event' | 'research' | 'achievement' | 'announcement'`, `featured: boolean`, `active: boolean`

The context key is `'adt-website-data'` in localStorage. Call `resetToDefaults()` from the admin dashboard to restore seed data.

## Bilingual support

Every user-facing string has an EN and TH variant. The `language` value from `useApp()` is `'en'` or `'th'`. Pattern:

```tsx
const { language } = useApp();
// inline
{language === 'en' ? 'English text' : 'ข้อความภาษาไทย'}
// from data model
member.name / member.nameTH
news.title / news.titleTH
```

English is the default language on load.

## Admin backoffice

- Route: `/admin`
- Password: `admin123` (checked against sessionStorage key `admin-auth`)
- All CRUD operations go through `useApp()` context methods: `addStaff`, `updateStaff`, `deleteStaff`, `addHighlight`, `updateHighlight`, `deleteHighlight`, `addNews`, `updateNews`, `deleteNews`
- Use `.admin-input`, `.admin-label`, `.admin-btn-primary`, `.admin-btn-secondary`, `.admin-btn-danger` CSS classes for form elements

## ScrollReveal usage

```tsx
<ScrollReveal delay={200} direction="up">
  <YourComponent />
</ScrollReveal>
```

Props: `delay` (ms), `direction: 'up' | 'left' | 'right' | 'none'`, `threshold` (0–1), `once` (default true).

## Adding static assets

Copy files from `asset/` to `public/` before referencing them. Use `next/image` with `unoptimized` for external image URLs, or without it for `public/` files.

## Rules

- Keep the dark tech aesthetic — no light backgrounds on public pages.
- All new content fields must be bilingual (EN + TH).
- Data changes go through the context API, never direct `localStorage` calls outside `AppContext.tsx`.
- `ProgramsSection` data is static (hardcoded in the component) — there is no admin UI for programs.
- Do not add a real backend or database without discussion; current persistence is localStorage only.

---
> Source: [cnacha-mfu/adt-website](https://github.com/cnacha-mfu/adt-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
