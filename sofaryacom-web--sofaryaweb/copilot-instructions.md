## sofaryaweb

> **sofarya** is a curated restaurant reservation platform for Istanbul, positioned to expand into a **multi-category Turkish premium experiences platform** (restaurants → wine experiences → equestrian → yacht → hammam, etc).

# sofarya — Project Context

## What this is

**sofarya** is a curated restaurant reservation platform for Istanbul, positioned to expand into a **multi-category Turkish premium experiences platform** (restaurants → wine experiences → equestrian → yacht → hammam, etc).

Goal: 200 restaurants × 200 reservations/day at maturity. Premium curated, not mass-market. Resy/Tock segment, **not** OpenTable.

## Read these first (project root)

- **IDEAS.md** — strategic decisions, scope, doctrine, rejected ideas, open questions
- **BUYUME-ASAMALARI.md** — phased GTM/growth strategy (positioning, channels, budgets, 30-day plan)

Both are kept up to date. When the user says "fikirler / ideas" → IDEAS.md. When "büyüme / growth / acquisition" → BUYUME-ASAMALARI.md.

## Brand Identity (quick reference)

- **Name**: `sofarya` (always lowercase). Domain: sofarya.com
- **Logo**: SVG component `<SofaryaLogo />` + static `/public/logo.svg` and `/public/logo-light.svg`. Fonts: Playfair Display (wordmark), Inter (tagline). Dot is `<circle cx=126 cy=34 r=3.5 fill=#C85A3D>`. Tagline `THIS WAY.` lives under wordmark.
- **Palette** (in `globals.css` via `@theme`):
  - cream `#faf8f5`, cream-soft `#f4f0e8`
  - ink `#1a1a1a`, ink-soft `#4a4540`, muted `#8b7e72`
  - terracotta `#c85a3d`, terracotta-dark `#a84a31`
  - border-soft `#e8e2d9`
- **Fonts**: serif `--font-playfair` (Playfair Display), sans `--font-inter` (Inter). All `font-serif` in Tailwind = Playfair Display.

## Tech Stack

- Next.js 15 (App Router) + React 19 + TypeScript
- Tailwind v4 (CSS `@theme` config — no tailwind.config.js)
- next/font: Playfair Display + Inter
- Leaflet + react-leaflet (map view)
- Auth: mock (localStorage), real Supabase planned
- AI Concierge: mock (keyword + synonym), real Claude Haiku planned
- WhatsApp: simulated, Twilio/Vonage planned
- Dev port: usually 3002 or 3003 (next available)

## Conventions

- **Language**: Turkish for all UI copy, microcopy, errors, labels. Code comments minimal.
- **No emojis in code** unless explicitly requested.
- **Don't add fake reviews / star ratings ever** (legal + brand doctrine — see IDEAS.md "⛔ Yapılmayacak").
- **Brand colors and font swaps need user confirmation** — don't change unilaterally.
- **`var` keyword** as object property breaks SWC parser — use `cssVar` instead (learned the hard way).
- **JSX text** containing `<` must be escaped as `&lt;` or wrapped in `{...}`.

## Current Build Status (2026-05-06)

### Pages built
- `/` — homepage with AI Concierge, Stories bar, "Niye sofarya?" band, Booking bar, Editorial Collections, curated grid (filters, meal toggle, list/map)
- `/restaurants/[slug]` — detail with editor note, chef section, picks, atmosphere, press, gallery, **"Sofarya Stüdyosu üretimi"** badge, ChefStory video card, sticky reservation form
- `/account` — profile, preferences (from onboarding), reservations (upcoming/past), favorites
- `/hakkimizda` — full About: niye varız, nasıl seçiyoruz, Sofarya Stüdyosu, Editör Defne, yakında, restoran ol, basın
- `/logo` — internal logo preview (NOT linked from main site, dev only)

### Data
**15 mock restaurants** in `src/lib/restaurants.ts`. Categories: 8 evening restaurants, 2 breakfast (Van Kahvaltı Evi, Privato Galata), 1 wine bar (Pera Şarap), 3 cocktail bars (Tütün Bar, Pera Roof, Yer Altı), 2 brunch/all-day (Demlik, Çiçek Sokak), Karadeniz Lokantası, Yedikule Bağ.

Each restaurant has: editor note, chef bio + portrait + quote, 3 editorial picks (signature dishes), atmosphere tags + details, 1-2 press quotes, video story, coords, tonightSlots, atmosphereTags, mealTypes (`'breakfast' | 'lunch' | 'dinner' | 'cocktail'`).

### Auth (mock)
- Phone + OTP (demo code: `123456`) or Google one-click
- After signup: 4-step onboarding survey (dietary, cuisines, neighborhoods, occasions)
- Stored in `localStorage`: `sofarya.user`, `sofarya.reservations`, `sofarya.favorites`, `sofarya.preferences`, `sofarya.onboarded`

### Key components
`HeaderClient`, `Footer`, `HomeExperience`, `WhySofarya`, `StoriesBar`, `EditorialCollections`, `RestaurantCard` (with time chips + heart), `QuickBookModal`, `BookingBar`, `FilterChips`, `MealTimeToggle`, `MapView` (dynamic), `AiSearch`, `AuthModal`, `OnboardingSurvey`, `ChefSection`, `PicksAndAtmosphere`, `PressSection`, `ChefStory`, `ReservationForm`, `SofaryaLogo`.

### Utilities
- `src/lib/restaurants.ts` — Restaurant type, MealType, slotsForMeal, mealOf, priceLabel, getRestaurant
- `src/lib/auth.tsx` — AuthProvider, useAuth, Preferences type
- `src/lib/search.ts` — keyword + synonym search (mock for AI)
- `src/lib/collections.ts` — 3 editorial collections (Boğaz'da bahar, Şefin gizli menüsü, İlk randevu)
- `src/lib/ics.ts` — calendar (.ics) file generator with VALARM

## Open Decisions (next session)

1. **Hero photo** — currently photo-1517248135467 (Lokanta Saray cover style). User may want own production photo eventually.
2. **Data model: Restaurant → Experience** — should generalize for multi-category (wine, equestrian, etc) now or after launch?
3. **Next feature priority** — Notify Me, real Claude AI, Sofarya Üyeliği tier, Supabase, restaurant admin panel, Hit List rotation system. User hasn't picked.
4. **Reklam bütçesi & timing** — paid acquisition strategy committed but budget/start not confirmed.
5. **Pilot 5 video çekimi** — recipe locked, but no production date yet. Lokanta Saray + Ocakbaşı Pera + Mavi Ev + Pera Şarap Evi + Yedikule Bağ.

## NOT yet built (from review checklist)

These were intentionally skipped in last session — pick up next session if user asks:
- Item 3: WhatsApp-first vaadi visible on site (currently brand says it but UI doesn't show it)
- Item 5: Editor Defne byline on every restaurant detail page (currently only in Editorial Collections)
- Item 9: Standalone /restoran-ol B2B page (currently part of /hakkimizda#restoran)

## Memory files

In `C:\Users\DELL\.claude\projects\c--Users-DELL-Desktop-Proje1-RESTAURANTtrip\memory\`:
- `MEMORY.md` — index
- `project_brand.md` — brand identity rules
- `project_overview.md` — product overview & tech decisions
- `project_video_production.md` — chef video recipe (5 questions, 60s structure)
- `project_strategic_pivot.md` — production house + multi-category platform pivots
- `project_growth_phases.md` — GTM phased plan (4 phases, ~₺4.7M / 12 months)
- `project_current_state.md` — site build snapshot (this file's twin)

## How to resume

If user says "where did we leave off" or starts fresh:
1. Read this file (CLAUDE.md)
2. Skim IDEAS.md "Sıradaki Yapılacaklar" + "Açık Kararlar"
3. Skim BUYUME-ASAMALARI.md "Aşama 0"
4. Ask user what to focus on, don't auto-pick.

If user says "show me what's there" / "siteyi göster":
- Open dev server (usually `npm run dev`, port 3002 or 3003)
- Pages: `/`, `/restaurants/lokanta-saray`, `/account`, `/hakkimizda`

---
> Source: [sofaryacom-web/sofaryaweb](https://github.com/sofaryacom-web/sofaryaweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
