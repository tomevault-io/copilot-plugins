## an-video-templates

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Remotion-based motion graphics pipeline for the Action Network. Takes article text or stats data → generates broadcast-quality animated video graphics (MP4s) via 12 reusable templates. Includes an AI-powered blueprint generation step (Claude API) and a companion design builder UI.

## Commands

```bash
npm install

# Remotion Studio — live preview with animation timeline
npm run dev              # → localhost:3000

# Template Builder — form-based designer, scaled previews, render command generator
npm run builder          # → localhost:3001

# Rendering
npx remotion render src/index.ts OddsCard out/my.mp4 --props='{"teamA":{"name":"KC","odds":"-180"}}'
npm run render:all                                    # batch render all compositions
npm run render:blueprint path/to/blueprint.json       # render from AI-generated blueprint

# AI blueprint generation
npm run generate -- --input article.txt              # from file
npm run generate:url -- https://example.com/article  # from URL (via Jina Reader)
cat article.txt | npm run generate                    # from stdin
```

No test runner or linter is configured.

## Architecture

### Data flow

```
Article/stats input
  → generate-blueprint.mjs (calls Claude API with full template reference)
  → out/blueprint.json (beats: narration + shot specs with compositionId + props)
  → render-blueprint.mjs (npx remotion render per shot)
  → out/beat-{id}-shot-{i}-{template}.mp4
  → assembled in video editor with voiceover
```

### Schema-driven design — the key architectural pattern

`src/lib/schemas.js` is the single source of truth for all template field definitions. Both the Remotion templates and the Builder UI import from it. Adding or changing a field means:
1. Update the schema in `schemas.js`
2. Update the template component to match
3. The Builder form auto-updates — no separate form code needed

### Template structure

Each template in `src/templates/` is a self-contained Remotion component:
- Exports a named React FC (e.g., `export const OddsCard`)
- Uses `useCurrentFrame()` + `useVideoConfig()` + `spring()` + `interpolate()` from Remotion for all animation
- Renders into `<AbsoluteFill>` with inline styles
- All props typed via interfaces in `src/lib/theme.ts`

All 16 compositions (some templates have variants, e.g. `ProbabilityViz` / `ProbabilityViz-Grid`) are registered in `src/Root.tsx` with `durationInFrames`, dimensions (1920×1080), and 30fps.

Sample data for all templates lives in `src/data/miracle-on-ice.ts` (1980 Olympics hockey) and is used as `defaultProps` in Root.tsx.

### Animation pattern

```typescript
const frame = useCurrentFrame();
const { fps } = useVideoConfig();
const progress = spring({ frame, fps, config: anim.springSmooth, durationInFrames: 15 });
// Stagger elements by adding offsets: spring({ frame: frame - 6, ... })
```

Spring configs and timing constants are in `src/lib/theme.ts` (`anim.springSnappy/Bouncy/Smooth`).

### Builder app (`builder/`)

Separate Vite app (port 3001) that reads `schemas.js` to dynamically generate forms. Provides specialized paste components (PasteStats, PasteTable, PasteLines, PasteKV, PastePoints) for tabular data entry and generates a ready-to-paste `npx remotion render` command.

### Blueprint generation (`scripts/generate-blueprint.mjs`)

Calls Claude Sonnet with a system prompt that includes the full template reference. The LLM structures content into 4–8 narrative beats, each with one or more graphic shots mapped to template compositionIds and props. The script validates JSON output and ensures script beats match shot beats. Key constraint: no invented data — fact-based only.

`render-blueprint.mjs` caps `durationInFrames` to each template's max to prevent LLM overreach.

## Design system (`src/lib/theme.ts`)

**Colors:** Dark broadcast palette — `#15100A` bg, `#D4A843` warm gold (green/positive), `#C8281E` archive red (negative), `#F0E5CC` aged paper cream for text.

**Typography:**
- `fonts.display` — League Gothic (headlines)
- `fonts.stats` — Bebas Neue (numbers, scores)
- `fonts.body` — Space Grotesk (labels, descriptions)
- `fonts.mono` — IBM Plex Mono (timestamps, attribution)

**Aesthetic:** Sharp corners (no border-radius), vintage broadcast feel. "ACTION" watermark bottom-right on every template via `styles.watermark`.

## Adding a template

1. Add schema to `src/lib/schemas.js` (SCHEMAS array)
2. Create `src/templates/TemplateName.tsx`
3. Add sample data to `src/data/miracle-on-ice.ts`
4. Register `<Composition>` in `src/Root.tsx`
5. Add to the system prompt in `scripts/generate-blueprint.mjs` so the LLM knows it exists

---
> Source: [petro438/an-video-templates](https://github.com/petro438/an-video-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
