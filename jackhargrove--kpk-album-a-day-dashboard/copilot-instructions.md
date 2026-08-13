## kpk-album-a-day-dashboard

> This is a React dashboard (dashboard.jsx) tracking progress through a 318-album version of the /mu/ essentials 2018 list. Two people — **Hargrove (Jack)** and **Aidan** — are listening to one album per day and rating them. The dashboard visualizes their progress, ratings, and preferences.

# /mu/ Essentials 2018 — Album a Day Dashboard

## Project Overview

This is a React dashboard (dashboard.jsx) tracking progress through a 318-album version of the /mu/ essentials 2018 list. Two people — **Hargrove (Jack)** and **Aidan** — are listening to one album per day and rating them. The dashboard visualizes their progress, ratings, and preferences.

The dashboard renders as a Claude artifact (JSX with Recharts). There is also an HTML export (`mu-essentials-dashboard.html`) for sharing, though it currently has issues on mobile Safari.

## File Structure

- `dashboard.jsx` — The main React component. This is the only file you need to edit.
- `mu-essentials-dashboard.html` — Standalone HTML export (needs rebuilding after JSX changes via TypeScript transpilation, see below)

## Data Model

Each album in the `ALBUMS` array has these fields:

```
{
  artist: "Artist Name",
  title: "Album Title",
  year: 1999,
  genre: "Indie / Alt",          // One of 11 genres (see GENRE_COLORS)
  section: "Modern",             // Original /mu/ list section: "/mu/-core", "Modern", "Classics", "Jazz", "Electronic", "Hip-Hop", "Metal"
  group: "Daniel Lopatin",       // OPTIONAL — groups related projects (e.g., Chuck Person + OPN = "Daniel Lopatin", Brian Eno solo + Eno & Byrne = "Brian Eno")
  listenOrder: 42,               // OPTIONAL — sequential order listened (1-212), only on listened albums
  rank: 7,                       // OPTIONAL — Hargrove's personal ranking (1=favorite, 212=least), only on listened albums
  tier: "SS",                    // OPTIONAL — Hargrove's tier: SS/S/A/B/C/D/F, only on listened albums
  aidanTier: "A",                // OPTIONAL — Aidan's tier, only on listened albums
  priorListen: true,             // OPTIONAL — Hargrove had heard this before the project
  aidanPrior: true,              // OPTIONAL — Aidan had heard this before the project
  listened: true                 // Whether this album has been listened to yet
}
```

## Genres (11 total)

Indie / Alt, Electronic, Experimental, Pop / Rock, Punk / Post-Punk, Hip-Hop, Jazz, Metal, Prog, Folk / Singer-Songwriter, Industrial

## Tier Breaks (Hargrove)

- SS: rank 1-13
- S: rank 14-41
- A: rank 42-79
- B: rank 80-135
- C: rank 136-174
- D: rank 175-199
- F: rank 200-212

Aidan has tiers but NO granular ranking. His tier sort is SS→S→A→B→C→D→F, alphabetical by artist within tiers.

## Viewer Tab System

The dashboard has a "Hargrove" / "Aidan" toggle. The `viewer` state controls which person's tier/rank/prior data is displayed. Key functions:

- `getTier(a)` — returns `a.tier` or `a.aidanTier` based on viewer
- `getRank(a)` — returns `a.rank` for Hargrove, `null` for Aidan

## Multi-Album Artist Ordering

Artists with multiple albums are ordered **chronologically** in the data array. The sort uses `_idx` (array position) as a tiebreaker so this ordering is preserved in the table and artist chart. This is intentional and carefully maintained — e.g., David Bowie is Ziggy (1972) → Low (1977) → Heroes (1977) → Blackstar (2016).

## Adding a New Album Listen

When a new album is listened to (album #213, #214, etc.):

1. Find the album in the ALBUMS array (it already exists with `listened: false`)
2. Add `listenOrder: 213` (or whatever the next number is)
3. Add Hargrove's `rank` and `tier` — this requires incrementing the rank of every album currently at or below the insertion point. E.g., if new album is rank 45, every album with rank >= 45 gets rank += 1
4. Also update Hargrove's tier breaks if the insertion shifts albums across tier boundaries
5. Add `aidanTier: "X"` with Aidan's tier
6. Set `listened: true`

## Genre Classification Principles

- Genre assignments are debated and deliberate; Jack has final say but welcomes pushback
- "Industrial" includes: NIN, Whitehouse, Sotos, Coil, Big Black (×2), Foetus, Daughters, Swans' Filth
- "Folk / Singer-Songwriter" abbreviates to "Folk / S-S" in the genre ranking chart
- "Punk / Post-Punk" abbreviates to "Punk / P-P" in the genre ranking chart
- Safe as Milk is Pop/Rock (not Experimental like Trout Mask Replica)
- NIN stays in Industrial even though it could be Metal
- Slint is Punk/Post-Punk (not Experimental, despite being the ur-post-rock album)

## Rebuilding the HTML Export

The HTML export requires pre-compiling JSX to plain JS (no Babel in browser — it's too slow on mobile). Steps:

```bash
# 1. Create a .tsx wrapper with type declarations
cat > dashboard.tsx << 'EOF'
declare var React: any;
declare var ReactDOM: any;
declare var Recharts: any;
const { useState, useMemo } = React;
const { BarChart, Bar, XAxis, YAxis, Tooltip, ResponsiveContainer, Cell, PieChart, Pie } = Recharts;
EOF

# Append JSX content (without import lines)
grep -v "^import " dashboard.jsx >> dashboard.tsx

# 2. Compile with TypeScript API
node -e "
const ts = require('typescript');
const fs = require('fs');
const src = fs.readFileSync('dashboard.tsx', 'utf8');
const result = ts.transpileModule(src, {
  compilerOptions: {
    jsx: ts.JsxEmit.React,
    target: ts.ScriptTarget.ES2020,
    module: ts.ModuleKind.None,
  }
});
fs.writeFileSync('dashboard.js', result.outputText);
"

# 3. Clean up module artifacts from the compiled JS and wrap in HTML
# Remove: "use strict", Object.defineProperty(exports...), exports.default = Dashboard
# Wrap in HTML with React + Recharts CDN scripts (no Babel needed)
```

Note: The HTML currently still has issues loading on mobile Safari. The Recharts CDN may not be loading properly. This is an open issue.

## Style Notes

- Dark theme (#0d0d0d background, monospace fonts)
- JetBrains Mono + Space Mono from Google Fonts
- All visualizations respond to the All/Listened/Remaining status toggle
- The table is sortable by any column; rank sort on Aidan's tab sorts by tier then alphabetically

---
> Source: [jackhargrove/KPK-Album-a-Day-Dashboard](https://github.com/jackhargrove/KPK-Album-a-Day-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
