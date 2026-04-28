## awesome-zk-proofs

> This is an interactive learning platform for Zero-Knowledge Proofs with two outputs:

# AGENTS.md - AI Agent Guidelines

## Project Overview

This is an interactive learning platform for Zero-Knowledge Proofs with two outputs:
1. **Web Application**: React-based visual journey map
2. **Awesome List**: Auto-generated README.md for GitHub

## Architecture

**Single Source of Truth**: `src/data/graphData.ts`
- All content (nodes, resources, descriptions) lives here
- Both the web app and README are derived from this file
- **NEVER edit README.md directly** - it is auto-generated

## Key Commands

```bash
npm run dev       # Start dev server on localhost:5173
npm run readme    # Generate README.md from graphData.ts
npm run build     # TypeScript compile + Vite production build
npm run deploy    # Build + deploy to GitHub Pages
```

## Adding/Editing Content

1. Edit `src/data/graphData.ts`
2. Run `npm run readme` to regenerate README.md
3. Test locally with `npm run dev`

### Data Structure

```typescript
interface NodeData {
  id: string;                    // Unique identifier
  title: string;                 // Display name
  category: string;              // Visual category
  description: string;           // One-line summary
  resources: Resource[];         // Learning materials
  x, y: number;                  // Unused (legacy)
  dependencies: string[];        // Prerequisite node IDs
}

interface Resource {
  title: string;
  author?: string;
  url: string;
  type: 'Article' | 'Video' | 'Book' | 'Course' | 'Tool' | 'Paper' | 'Podcast' | 'Newsletter' | 'Community' | 'Program' | 'Organization';
  rating: number;                // 1-10 scale, used for sorting (not displayed)
  description: string;
}
```

## Important Files

| File | Purpose |
|------|---------|
| `src/data/graphData.ts` | **Single source of truth** for all content |
| `scripts/generate_awesome_list.mjs` | README generation script |
| `src/components/JourneyMap.tsx` | Main UI with hardcoded chapter structure |
| `index.html` | HTML template with CSP policy |
| `vite.config.ts` | Build config (base path: `/awesome-zk-proofs/`) |

## Gotchas

1. **Hardcoded Node References**: `JourneyMap.tsx` has hardcoded node IDs in `chapters`. Changing node IDs requires updating both files.

2. **README is Generated**: Never manually edit README.md - run `npm run readme` after editing graphData.ts.

3. **CSP Policy**: Located in `index.html`. Must include domains for external resources (fonts, analytics, etc.).

4. **Deduplication**: README generator deduplicates by URL/title.

5. **Base Path**: App is served from `/awesome-zk-proofs/` - all assets assume this path.

## Content Security Policy

The CSP in `index.html` controls which external resources can load. When adding external resources:
- Fonts: Add to `style-src` and `font-src`
- Scripts: Add to `script-src`
- Images: Add to `img-src`
- API calls: Add to `connect-src`

## Deployment

- GitHub Pages via custom domain
- CNAME file in `public/`
- Production URL: https://floatingpragma.io/awesome-zk-proofs/

---
> Source: [muellerberndt/awesome-zk-proofs](https://github.com/muellerberndt/awesome-zk-proofs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
