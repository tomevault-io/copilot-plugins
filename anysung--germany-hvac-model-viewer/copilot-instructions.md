## germany-hvac-model-viewer

> These rules are **confirmed and locked**. Apply them to all future changes unless explicitly overridden by the user.

# German Heat Pump Database — Layout & Alignment Rules

These rules are **confirmed and locked**. Apply them to all future changes unless explicitly overridden by the user.

---

## 1. Filter Section (HeatPumpApp.tsx)

### Row 1 — Manufacturer
- Full-width single panel: `bg-white px-3 py-2 rounded-lg shadow-sm border border-gray-200`
- Label: `text-[10px] font-bold text-gray-400 uppercase tracking-wider mb-1`
- Badges: horizontal wrap `flex flex-wrap gap-1.5`

### Row 2 — Capacity | Installation Type | Refrigerant Type
- 3-column grid with **unequal widths**: `gridTemplateColumns: '5fr 4fr 2.5fr'` (inline style, not Tailwind grid)
- All three panels use the same panel style as Row 1
- **Capacity** (5fr): 4 badges — `4 kW ~ 7 kW`, `8 kW ~ 10 kW`, `11 kW ~ 12 kW`, `13 kW ~ 17 kW` — must all fit in **one row**
- **Installation Type** (4fr): 2 badges — `Monoblock`, `Split` — must all fit in **one row**
- **Refrigerant Type** (2.5fr): 3 badges — `🌿 R290`, `R32`, `R410A` — must all fit in **one row**
- Refrigerant filter logic: **contains** (`item.refrigerant.includes(value)`), not exact match
- Installation Type filter logic: matches `item.installation_type` directly (exact match)

### Section spacing
- `mb-2 space-y-1.5` between rows — keep compact, no excess vertical padding

### Active Filters Bar
- Shown only when at least one filter is active
- Clear All button: `bg-red-600 hover:bg-red-700 text-white text-xs font-bold rounded border border-red-700 shadow-sm` with `ml-auto`
- Active filter chips: yellow for search query, blue for brand, green for refrigerant

---

## 2. Results Table (ResultsTable.tsx)

### Column Order (left → right)
| # | Column | Sticky | Notes |
|---|--------|--------|-------|
| 1 | Manufacturer | ✅ Left-sticky | Center aligned (header + content) |
| 2 | Installation Type | ✅ Left-sticky | Monoblock = orange badge, Split = purple badge |
| 3 | Model | ✅ Left-sticky | Max 35 chars, truncate with `…`, full on hover |
| 4 | Capacity | — | 2-line split at `' ('` or `';'` |
| 5 | Refrigerant | — | R290 (contains) → `🌿` green; others gray |
| 6 | Dimensions | — | 2-line split at `'; '` or `';'` |
| 7 | COP | — | Blue header bg, 2-line split at `'; '` |
| 8 | SCOP | — | Blue header bg, 2-line split at `'; '` |
| 9 | Noise | — | Blue header bg, 2-line split at `';'` |
| 10 | Weight | — | Purple header bg, extracted from `others` field |
| 11 | Price | — | Green header bg, 2-line split at `' ('` |
| 12 | Others | — | Remaining after weight extraction, `min/max-w-[400px]` |

### Sticky Column Rules
- Manufacturer: `sticky left-0 z-20` (body), `sticky top-0 left-0 z-40` (header)
- Type: `sticky left-[mfrWidth] z-20/40` (measured via `useLayoutEffect`)
- Model: `sticky left-[mfrWidth+typeWidth] z-20/40` (measured via `useLayoutEffect`)
- All sticky cells: **solid white** background (`bg-white`) — never transparent/semi-transparent
- Non-sticky header cells: `sticky top-0 z-30`

### Font Sizes
- Header (TH): `text-[12px]`
- Body (TD): `text-[13px]`
- Badges / second lines / small labels: `text-[11px]`
- Others column font: same as TD (`text-[13px]`)

### Two-Line Display (TwoLine component)
- Line 1: normal weight
- Line 2: `text-[11px] text-gray-400 mt-0.5`
- All cells center-aligned within `TwoLine`

### Scroll Control Bar (top of table)
- Blue progress bar showing horizontal scroll position
- Left `←` button (disabled when at start)
- Right `→ more` button with blue bg when more content exists (disabled when at end)
- Right-edge fade gradient overlay when scrollable

### Table Dimensions
- `max-h-[70vh]` with `overflow-y-auto` for vertical scroll
- `overflow-x-auto` for horizontal scroll
- Striped rows: even=`bg-white`, odd=`bg-gray-50/50`, selected=`bg-blue-50`

### R290 Green Icon Rule
- Condition: `item.refrigerant.includes('R290')` — **contains**, not exact match
- Display: `🌿` prefix + `text-green-600 font-bold`
- Applies to values like `R290(estimated)`, `R290(likely)`, etc.

### Weight Extraction Logic
- Try labelled: `/(?:weight|wt\.?|gewicht)[:\s]+(\d+(?:[.,]\d+)?\s*kg)/i`
- Fallback: standalone `/\b(\d+(?:[.,]\d+)?\s*kg)\b/i`
- Remaining text after extraction → Others column
- If no weight found: display `—`

---

## 3. Market News Image Rules (NewsView.tsx + index.js)

### Core Rule
- **Every news article must always display an image** — never leave the image slot empty.
- **Never let AI (Gemini) generate or hallucinate image URLs** — it repeats or fabricates broken URLs.
- Images are assigned by **keyword matching** against the article title + summary.

### Image Assignment Logic
- `selectNewsImage(title, summary)` matches keywords (case-insensitive) in priority order:

| Priority | Category key | Trigger keywords |
|---|---|---|
| 1 | `subsidy` | bafa, beg, subsidy, funding, grant, zuschuss, kfw, förder |
| 2 | `government` | parliament, bundestag, bundesrat, minister, geg, regulation, gesetz, law |
| 3 | `solar` | solar, photovoltaic, pv, renewable, wind, erneuerbar |
| 4 | `technology` | r290, r32, refrigerant, cop, scop, efficiency, innovation |
| 5 | `installation` | install, installer, montage, handwerk, technician |
| 6 | `market` | market, sales, statistics, trend, bwp, report, growth, demand |
| 7 | `energy` | energy, electricity, power, grid, strom, tariff |
| 8 | `house` | house, home, building, residential, renovation, retrofit |
| 9 | `heatpump` | heat pump, wärmepumpe, hvac, viessmann, vaillant, stiebel… |
| default | `heatpump` | (no match) |

### Curated Unsplash Image URLs (open-source, no API key needed)
```
heatpump:     photo-1621905251189-08b1059efa82
subsidy:      photo-1554224155-6726b3ff858f
house:        photo-1570129477492-45c003edd2be
government:   photo-1555900234-35b55afe19df
solar:        photo-1509391366360-2e959784a276
technology:   photo-1518770660439-4636190af475
installation: photo-1504307651254-35680f356dfd
market:       photo-1551288049-bebda4e38f71
energy:       photo-1473341304170-971dccb5ac1e
```
URL format: `https://images.unsplash.com/photo-{ID}?auto=format&fit=crop&q=80&w=600`

### Both sides must be in sync
- `NEWS_IMAGES` + `NEWS_IMAGE_RULES` + `selectNewsImage()` exist in **both** `index.js` (Cloud Function) and `NewsView.tsx` (frontend fallback).
- Cloud Function assigns `imageUrl` at write time; frontend uses `onError` fallback if URL breaks.
- When adding a new image category: update **both** files.

---

## 4. General Code Rules

- **Do not change column order** without explicit user instruction
- **Do not change sticky column behavior** (Manufacturer, Type, Model always sticky)
- **Do not make sticky backgrounds transparent** — always solid white
- **Do not widen/narrow Others column** from `min-w-[400px] max-w-[400px]` without explicit instruction
- **Refrigerant filter** always uses `.includes()` contains logic, never exact match
- **Installation Type filter** matches `installation_type` directly; "Set" products match both Monoblock and Split
- **Filter grid proportions** (`5fr 4fr 2.5fr`) must keep all badges in single row — verify before changing
- Build command: `export PATH="/Users/christophersung/.nvm/versions/node/v20.19.6/bin:$PATH" && npm run build`
- Deploy command: `firebase deploy --only hosting`

---
> Source: [anysung/germany-hvac-model-viewer](https://github.com/anysung/germany-hvac-model-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
