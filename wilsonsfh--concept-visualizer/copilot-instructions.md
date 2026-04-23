## concept-visualizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server at http://localhost:3000
npm run build    # Production build
npm run lint     # ESLint
```

No test suite is configured. There is no `npm test` command.

## Environment

Requires a `.env.local` file with:
```
GROQ_API_KEY=your_key_here
UPSTASH_REDIS_REST_URL=your_upstash_url
UPSTASH_REDIS_REST_TOKEN=your_upstash_token
```

## Architecture

The app is a Next.js 15 (App Router) AI-powered concept visualizer. A user submits a plain-language query → the API calls Groq → returns a structured JSON visualization → rendered as an interactive React Flow graph with step-by-step animation.

### Data flow

```
User query
  → POST /api/visualize
    → Groq (llama-3.3-70b-versatile) via Vercel AI SDK
    → JSON extracted + validated by Zod (VisualizationSchema)
  → VisualizationResponse stored in Zustand (useVisualizationStore)
    → transformer.ts: maps nodes/edges + per-step active state to React Flow types
    → layout.ts: Dagre auto-layout assigns positions
  → VisualizationCanvas renders React Flow
  → StepControls + StepDescription consume currentStepIndex from store
```

### Key boundaries

- **`src/lib/ai/schema.ts`** — Zod schema is the contract between Groq output and the app. Nodes: 5–15, steps: 3–10.
- **`src/lib/claude/prompt-builder.ts`** — System prompt that instructs Groq to produce the schema-conforming JSON.
- **`src/lib/visualization/transformer.ts`** — Converts `VisualizationResponse` + current `AnimationStep` into React Flow `Node[]`/`Edge[]`. Active/inactive visual state is computed here (not in components).
- **`src/lib/store/visualization-store.ts`** — Single Zustand store owns all visualization state including playback.

### Node types

Three custom React Flow node types, each with distinct visual style:
- `ConceptNode` — blue, concepts/components
- `DataNode` — green, data/state
- `DecisionNode` — amber, decision points

All are registered in `src/components/visualization/nodes/index.ts` and passed to React Flow via `nodeTypes`.

### Styling

Tailwind CSS v4, dark terminal aesthetic. Fonts: JetBrains Mono (monospace) + IBM Plex Sans (sans), loaded via `next/font` in `layout.tsx`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wilsonsfh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
