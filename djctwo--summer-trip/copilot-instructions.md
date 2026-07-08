## summer-trip

> Melissa's Jul 3–12, 2026 trip dashboard. Interactive single-file HTML with Leaflet.js maps, password-gated, dark theme.

# Summer Trip Dashboard

## Overview
Melissa's Jul 3–12, 2026 trip dashboard. Interactive single-file HTML with Leaflet.js maps, password-gated, dark theme.

## Repository
- **Path**: `/Users/aaxis/Documents/GitHub/summer-trip`
- **Remote**: `https://github.com/djctwo/summer-trip`
- **Branch**: main
- **Live site**: `https://djctwo.github.io/summer-trip/`

## File Locations
- **Working file (edit this)**: `/Users/aaxis/Documents/GitHub/germany/travel-dashboard.html`
- **Deployed copy**: `index.html` in this repo → GitHub Pages
- After editing the working file, copy to this repo's `index.html` and push to deploy
- Both files must stay in sync

## Dashboard Details
- **Password**: hashed with SHA-256 (`hotcheetos123!@#`)
- **Theme**: Costa Azul (coastal European warmth — Playfair Display + Outfit fonts, terracotta/ocean blue/olive/golden palette)
- **Design context**: `.impeccable.md` in both repos defines brand personality (Warm, Excited, Polished), color palette, typography, spacing
- **Day/Evening modes**: Toggle in topbar with localStorage persistence. Day = light/airy (#f8fbfd bg), Evening = warm/intimate (#1e2a3a bg)
- **Maps**: Leaflet.js via CDN, dark CartoDB tiles
- **Single-file**: ~4120 lines, all JS/CSS embedded
- **Evening mode coverage**: 100+ `.eve-mode` rules cover all views (Overview, Rec, Plan tabs, footer, passport stamps)
- **Color palette**: Strictly Costa Azul — no Tailwind colors, all route/timeline/text colors from `.impeccable.md` palette
- **Animations**: Scroll-triggered section reveals (IntersectionObserver), view fade transitions, password gate fade-out, tab indicator, map marker hover scale, shimmer pause on tab hidden
- **Accessibility**: `prefers-reduced-motion` global override, WCAG AA contrast ratios, 44px minimum touch targets

## Views / Tabs
Three view modes managed by `hideAllViews()`:
1. **Overview** (default landing) — hero, stats bar, "What's Locked In", 10-day calendar strip, plan cards grid, budget chart (Chart.js bar), chill chart (Chart.js doughnut), plan comparison picker with dynamic analysis, key decisions
2. **Plan Content** (8 plan tabs) — trip overview summary, route map, day-by-day itinerary, weather cards, destination cards, travel notes, packing checklist, booking links, emergency info, hotel recommendations (Porto), share/copy itinerary
3. **Recommended Pick** — Porto recommendation with pros/cons, why-Porto rationale

## Current Plans (8 tabs)
All plans include **Neuschwanstein Castle + Munich** as a locked 2-day block (must-do's).
4th of July must be in a non-Germany country.

| # | Plan Key | 4th of July | Castle+Munich | Extra |
|---|----------|-------------|---------------|-------|
| 1 | primary | Paris overnight (TGV) | Jul 7-8 | Amsterdam 1 night |
| 2 | balanced | Paris day trip (TGV) | Jul 6-7 | Amsterdam 2 nights |
| 3 | chill | Paris overnight (TGV) | Jul 8-9 | 4 KL chill days |
| 4 | minimal | Strasbourg day trip | Jul 6-7 | Amsterdam 2 nights |
| 5 | provence | Arles 2 nights (fly LH) | Jul 8-9 | 3 KL chill days |
| 6 | milan | Milan 3 nights (Ryanair €63/pp) | Jul 8-9 | 2 KL chill days |
| 7 | porto | Porto 2 nights (Ryanair €229/pp) | Jul 7-8 | 4 KL chill days |
| 8 | palma | Palma 3 nights (Ryanair €101/pp) | Jul 8-9 | 2 KL chill days |

## Features Per Plan Tab
- **Trip Overview Summary**: paragraph-style overview (~900-1050 chars)
- **Route Map**: Leaflet.js with plan-specific markers and routes
- **Day-by-Day Itinerary**: timeline with destinations, drives, activities
- **Weather Cards**: avg temp, rain chance, description per city
- **Destination Cards**: emoji hero + highlights per destination
- **Travel Notes**: plan-aware (only shows info for cities in that plan)
- **Packing Checklist**: base items + conditional items per plan, localStorage persistence
- **Booking Links**: plan-specific booking resources
- **Emergency Info**: embassy, insurance, emergency numbers
- **Hotel Recommendations**: Porto (6 hotels) + Munich (3 hotels) with Booking.com search-based links (direct URL slugs were broken — switched to `searchresults.html?ss=` format)
- **Share/Copy Itinerary**: Clipboard API, share toast notification

## Plan Comparison Picker
- Interactive pill selector for comparing 2+ plans side-by-side
- Compares: duration, hotel nights, transport, destinations, budget, chill days
- Dynamic analysis summary below the table with best-for recommendations

## Locations on Map
KL, Frankfurt (FRA), Strasbourg, Paris, Neuschwanstein, Munich, Amsterdam, Marseille (MRS), Arles, Luxembourg (LUX), Porto (OPO), Milan Bergamo (BGY), Palma (PMI), Frankfurt Hahn (HHN)

## Trip Fixed Points
- **Arrive**: Jul 3 (Fri), FRA 11:15 AM, UA1565/UA8879
- **Depart**: Jul 12 (Sun), FRA 9:45 AM, LH100/UA8918
- **Home base**: Kaiserslautern (KL)
- **Vehicle**: Tesla Model 3
- **Who**: Melissa visiting Doug

## Technical Architecture
- `renderPlan()` function wrapped twice: `_origRenderPlan` (view toggle + map fix) → `_origRenderPlan2` (new features rendering)
- `hideAllViews()` manages visibility of `overview-view`, `rec-view`, `plan-content`
- `map.invalidateSize()` called before `fitBounds()` to fix hidden container sizing
- `sessionStorage` for password unlock persistence across page navigations
- `localStorage` for packing checklist state persistence
- Chart.js for budget bar chart and chill days doughnut chart in Overview

## Conventions
- Update CLAUDE.md and Session_History continuously
- Run `/save` before `/compact`
- Day-of-week: Jul 3=Fri, Jul 4=Sat, Jul 5=Sun, Jul 6=Mon, Jul 7=Tue, Jul 8=Wed, Jul 9=Thu, Jul 10=Fri, Jul 11=Sat, Jul 12=Sun

---
> Source: [djctwo/summer-trip](https://github.com/djctwo/summer-trip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
