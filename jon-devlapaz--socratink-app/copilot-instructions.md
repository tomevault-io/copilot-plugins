## socratink-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**socratink** is an MVP-stage hosted learning product, not just a standalone prototype. This repo contains both the deployable app and the `learnops/` prompt assets that drive extraction and drill behavior.

Canonical repo identity, workflow ownership, and agent constraints live in `AGENTS.md` and `docs/project/state.md`. This file is implementation-oriented and should stay focused on runtime shape and local development details.

Current product framing: AI should remove prep friction and increase truthful retrieval reps without replacing the learner's generation step.

The UI maps internal JS state names to visual indicators (colored `.concept-dot` elements in the sidebar list, plus dark mode for `hibernating`):
- `growing` — active/structurally sound
- `fractured` — misconception detected, needs repair
- `hibernating` — consolidating; triggers `body.night` dark mode
- `actualized` — consolidated/mastered

> Note: `instantiated` still exists in the state machine for backward compatibility with old localStorage data, but new concepts are never created in that state.

## Running the Project

No build step. The FastAPI backend serves frontend files from `public/`:

```bash
uvicorn main:app --reload
# then open http://localhost:8000
```

## Architecture

The app is pure vanilla HTML/CSS/JavaScript with no build step and no frameworks. Frontend lives in `public/`, backend at root.

| File | Role |
|------|------|
| `public/index.html` | Shell: HTML structure + script/style tags |
| `public/styles.css` | CSS entry — `@import`s from `public/css/` |
| `public/js/app.js` | Main `App` IIFE — all UI logic, state machine, data store |
| `public/js/graph-view.js` | Cytoscape-based knowledge graph; derives `locked` / `drilled` / `solidified` node states from the knowledge map |
| `public/js/ai_service.js` | `AIService.generateKnowledgeMap()` — calls Python backend |
| `public/js/store.js` | localStorage CRUD, concept state definitions, transient content store |
| `public/js/bus.js` | Pub/sub event emitter for cross-section communication |
| `public/js/dom.js` | DOM element references |
| `public/js/geo.js` | Crystal polygon coordinates + easing utilities |
| `public/js/morph.js` | Crystal shape morphing engine (requestAnimationFrame) |
| `main.py` | FastAPI backend — `/api/extract`, `/api/drill`, static file serving |
| `ai_service.py` | Gemini API calls for extraction and Socratic drill |
| `api/index.py` | Vercel serverless entry point (imports `main.py`) |

**Script load order in `index.html`:** `ai_service.js` (global) → `app.js` (ES module)

**Key patterns in `app.js`:**
- **`setState(state)`** — central function that drives all UI transitions; calls `applyControlsForState()` at its end — this is the *only* place controls are updated (no manual `showControls`/`setButtons` calls elsewhere)
- **`applyControlsForState(state, concept)`** — owns all button visibility/enabled state for a given state
- **`setButtons(extract, drill)`** — two-parameter helper (extract-enabled, drill-enabled); `btn-present` was removed
- **CSS variables** — `--bg`, `--primary`, `--danger`, `--success` etc. for theming; `hibernating` state triggers a dark mode swap via `body.night`
- **24-hour timer** — `setInterval`-based countdown; a hidden dev-skip button (top-right corner, click to reveal) fast-forwards it for demos
- **Drill UI** — `#drill-ui` panel with `#chat-history` / `#chat-input` for chat-style Socratic drilling
- **Data persistence** — `localStorage` keys `learnops_concepts` (array of concept objects) and `learnops_active` (selected concept ID); max 4 concepts
- **Knowledge map as source of truth** — drill outcomes are patched into `concept.graphData` and the graph derives visual state from `drill_status` / `gap_type`, rather than treating Cytoscape state as authoritative
- **Transient content store** — `contentStore = new Map()` (module-level) holds full text keyed by concept ID; does not survive page reload (intentional)

**`app.js` is organized into 16 numbered sections** (see the table of contents comment near line ~25):
1. DOM references
2. Pub/sub `Bus` — lightweight event emitter used for cross-section communication
3. `GEO` — polygon coordinate arrays (5 states × 8 polygons) for the SVG crystal shapes
4. `STATES` — display config (title/desc text per state)
5. Coordinate utilities + easing
6. `MorphEngine` — shared `requestAnimationFrame` loop; morphs polygon coordinates between states
7. Transition animation helpers (`playAnim`)
8. Grid rendering — builds tile + crystal SVG content
9. Data store — `localStorage` CRUD helpers + `_readFile()` shared file-reading helper
10. Drawer UI
11. Concept list render
12. CRUD — `startAddConcept()` (content-first creation form), `renderAddTrigger()`, `deleteConcept()`
13. `setState()` + `applyControlsForState()`
14. Pipeline handlers — `extract()`, `showContentOverlay()`, `hideContentOverlay()`, `drill()`, `drillPass()`, `drillFail()`, `consolidate()`, `completeConsolidation()`, `fastForward()`
15. Timer
16. Init / restore

**`graph-view.js`** — exports `mountKnowledgeGraph({ container, detailEl, rawData, onNodeSelect })` and `transformKnowledgeMapToGraph(rawData)`. Cytoscape-based graph with solar orbit layout, ambient float animation, and epistemic node states:
- `locked` — not yet available
- `drilled` — attempted but not solid
- `solidified` — verified understanding

Cluster state is derived from subnode drill outcomes. The graph can live-sync from updated knowledge-map data via `syncFromKnowledgeMap(rawData, activeNodeId)`. Also exports `escHtml()` (shared HTML-escape utility).

**`ai_service.js`** — `AIService.generateKnowledgeMap(rawText, onProgress)` — calls the Python backend's `/api/extract` endpoint. Returns the extracted knowledge map JSON.

**CSS is split into modular files in `public/css/`:**
- `variables.css` — CSS custom properties (colors, spacing, radii, shadows)
- `base.css` — Reset, typography, body layout
- `crystal.css` — Crystal SVG styles, state-based color transitions, animations
- `components.css` — Buttons, drawer, concept list, drill UI, settings panel, creation form
- `layout.css` — Hero card, map view, graph layout, mobile breakpoints

## External Integrations & Configuration

Two settings are required for AI extraction — both configured via the **Settings** panel (sidebar nav → Settings):

| Setting | Storage | Key |
|---------|---------|-----|
| Gemini API key | `localStorage` | `gemini_key` |
| Staging directory handle | IndexedDB | `staging_dir_handle` |

The Gemini model is `gemini-2.5-flash` (constant `MODEL` in `ai_service.py`). The API key is resolved server-side from `GEMINI_API_KEY` env var or the client-provided key in localStorage via `_get_client()`.

The extraction system prompt is loaded by the Python backend from `learnops/skills/learnops-extract/extract-system-v1.txt`. The drill system prompt is loaded from `learnops/skills/learnops-drill/SKILL.md`.

`/api/drill` now uses a structured request/response contract:
- request includes `node_id`, `node_label`, `node_mechanism`, `knowledge_map`, `messages`, `session_phase`, `probe_count`, `nodes_drilled`, `session_start_iso`
- response includes `agent_response`, `classification`, `gap_description`, `routing`, `node_id`, `probe_count`, `nodes_drilled`, `session_terminated`, `termination_reason`

Known design debt: `node_mechanism` still travels from browser to backend on each drill request. There is a TODO in `main.py` to resolve the mechanism server-side from `knowledge_map` + `node_id` instead.

## Content-First Concept Creation

Concepts are created via a form embedded in the drawer (`startAddConcept()`). The form collects content first, name second, and creates the concept directly in `growing` state — bypassing `instantiated` entirely.

**Flow:**
```
Drawer → paste/upload content → type name → "Add Concept →" → concept born in 🌱 growing
```

**`startAddConcept()` form features:**
- Two tabs: **Text** (textarea with cycling placeholder) and **File** (.txt/.md/.pdf upload with drag-and-drop)
- "⌘ Paste from clipboard" button in the Text tab (`navigator.clipboard.readText()` with `execCommand` fallback)
- "Add Concept →" stays disabled until both content and name are provided
- On submit: stores full text in `contentStore`, persists `contentPreview` (first 500 chars) + `contentType` + `contentFilename` to localStorage, creates concept in `growing` state

**`_readFile(file, onSuccess, onError)`** — shared helper used by both `startAddConcept` and `showContentOverlay`:
- Validates extension (.txt/.md/.pdf) and size (≤2MB)
- PDF: ArrayBuffer → BT/ET regex extraction; fallback message if <50 chars extracted
- Text: `FileReader.readAsText()`

## Three-Stage Pipeline

1. **Ingest** (at concept creation) — user provides raw content (paste or file upload); concept is born in `growing` state with content stored
2. **Drill** — Socratic stress-testing tied to one graph node at a time. Drill outcomes are persisted into the knowledge map and projected onto the graph as `locked` / `drilled` / `solidified`
3. **Consolidate** — 24-hour sleep-gated hibernation; timer expires → concept becomes `actualized` 💎

> The "Extract" step (clicking "1. Extract" on a card) still exists as a fallback for legacy `instantiated` concepts in old localStorage data. New concepts never need it.

The biological constraint of hibernation-over-death is intentional: concepts go dormant rather than disappear, reducing shame spirals for learners with executive function variability.

## API Docs — Use chub Before Touching AI Code

The Python backend calls the Gemini API via the `google-genai` SDK. Before editing `ai_service.py`, fetch current docs with the `chub` CLI to avoid stale API assumptions:

```bash
chub get gemini/genai --lang py         # main Gemini Python reference
chub get gemini/deep-research --lang py # if working on any research-oriented extract flow
```

Common triggers:
- Changing the model constant `MODEL` → verify the model ID is still valid
- Modifying structured output or response schema enforcement → verify `response_schema` / `response_mime_type`
- Modifying retry or error handling → verify `google.genai.errors.APIError`
- Changing prompt construction or client setup → verify the current `google-genai` Python API shape

If you discover a gotcha during work (e.g. a streaming quirk, deprecated param), annotate it so future sessions start smarter:
```bash
chub annotate gemini/genai "your note here"
```

## learnops/ Agent Skills

The `learnops/` directory contains Claude Agent Skills for each pipeline stage. Stage 1 (Extract) is partially wired to the UI:

- `learnops/skills/learnops-extract/` — Stage 1. `extract-system-v1.txt` is loaded by `ai_service.py` as the Gemini system prompt during extraction. `SKILL.md` and `references/` contain the full cognitive processing pipeline spec.
- `learnops/skills/learnops-drill/` — Stage 3. `SKILL.md` is loaded by `ai_service.py` as the system prompt for Socratic drill chat. Wired to the UI via `/api/drill`.
- `learnops/skills/learnops-present/` — Stage 2. Standalone skill spec; not yet wired.
- Stage 4 (Consolidate) — designed but not yet built.

`learnops/tools/get_transcript.py` — CLI utility to fetch YouTube transcripts as text files (input for Stage 1). Requires `youtube-transcript-api`, `yt-dlp`, `pyperclip`, `rich`.

## Drill-Graph Context Hub

Before changing drill UX, graph UX, or Stage 3 prompt/routing behavior, read [docs/drill/graph-invariants.md](./docs/drill/graph-invariants.md).

That document defines the product invariants for drill-graph lockstep, the dungeon-room mental model, the epistemic state model (`locked` / `drilled` / `solidified`), cluster derivation rules, and the rules for when graph unlocks are allowed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-devlapaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
