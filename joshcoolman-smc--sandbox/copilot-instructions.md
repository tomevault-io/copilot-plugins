## sandbox

> Next.js design experiments sandbox. Each experiment is a self-contained route.

Next.js design experiments sandbox. Each experiment is a self-contained route.

## Commands

```bash
npm run dev      # Dev server :3000
npm run build    # Production build
```

## Frequent Workflows

**Quick capture (most common):**
- `/note` -- sticky note, instant capture
- `/link` -- save a link with comment, auto-thumbnails
- `/blog-post` -- draft from conversation context

**Design experiments:**
- `/sketch` -- rapid prototype (page.tsx + styles.css)
- `/ship-experiment` -- screenshot, gallery, README, commit, push

## SEO Checklist

After adding any new route or content section, touch these files:
- `app/sitemap.ts` -- add new routes to `staticRoutes` (dynamic routes like blog/experiments are auto-discovered)
- `public/llms.txt` -- concise entry
- `public/llms-full.txt` -- expanded entry with description
- `README.md` -- if it's a new section (not just a new item within existing content)
- `docs/01-progress.md` -- notable changes

Metadata: each page.tsx should export `metadata` with title + description. The root layout has OG/Twitter defaults.

## Structure

```
app/design-experiments/
├── page.tsx                       # Gallery
└── [experiment]/
    ├── page.tsx                   # Experiment page
    ├── styles.css
    ├── components/               # Extract when >500 lines
    ├── hooks/
    └── data/

app/(blog)/
├── blog/                          # Blog index and posts
├── recommended/                   # Link Worthy page
│   ├── items/                     # Individual link .md files
│   └── loadRecommended.ts         # Auto-fetches thumbnails at build
└── notes/                         # Sticky note .md files
```

## Component Extraction

Keep inline under 300 lines. Extract to `components/` when file exceeds 500 lines or component has a reusable interface.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshcoolman-smc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joshcoolman-smc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
