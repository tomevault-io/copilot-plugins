## coldpress

> Operating instructions for producing whimsical, retro-European interactive children's travel scrapbooks/activity journals. One brief in → one finished, print-ready scrapbook out.

# P8 — Children's Travel Scrapbook Factory

Operating instructions for producing whimsical, retro-European interactive children's travel scrapbooks/activity journals. One brief in → one finished, print-ready scrapbook out.

This file is **runtime spec**, not status. PARA status, Hemingway bridge, and dated notes live in sibling files.

---

## Status

- **Active book:** `books/hungary-2026/`
- **Deadline:** 2026-06-01 (print lead time before Jun 6 departure)
- **Pipeline stage:** ✅ Spec locked → ✅ Research locked → ✅ Editorial plan locked → ✅ Copy locked → ✅ Layouts locked → ✅ Illustrations locked → **→ Affinity assembly (next)**
- **Recraft credits:** ~2,480 / 8,000 used

## Hemingway Bridge

2026-05-07 [21:00] · Thursday — huge production day. All content tracks locked for `hungary-2026`. Spec, research, red-flag decisions, visual decisions, editorial plan (38pp: 4/15/8/8/3 split), copy (7 files, Hungarian throughout), layout specs (all sections + cover + sticker sheets), and 37 illustration assets are done. Recraft drama: trained custom style (`c9b1c5bb-…`, V3) underperforms V4 base for most assets — locked feedback memory. V3 kept only for 6 strong Budapest landmarks. 2,480/8,000 Recraft credits used. · Next: Canva Producer sub-agent — assemble book from locked assets. Start with Intro section (pp.1–4) to validate the full-bleed-to-Canva-frame workflow before full run. Canva MCP tools must be working: verify `mcp__canva__*` connection before firing the sub-agent. Deadline Jun 1 is 25 days out — Canva assembly is the only remaining track. · Blockers: none structural. If Canva MCP is unavailable, escalate immediately — this is a hard dependency for the final production phase.

2026-05-08 [late] · Switched from Canva to Affinity for the assembly track. Built affinity-design skill at .claude/commands/affinity-design/. Producer agent renamed scrapbook-canva-producer → scrapbook-affinity-producer; designer's Canva MCP tools swapped for Affinity. Next: dry-run the new Producer on the Intro section (pp.1–4) before full book.

---

## 1. Mission

Take a creative brief for a children's travel scrapbook and produce a print-ready interactive activity journal through a coordinated multi-agent pipeline. The system must generalise across briefs — no brief-specific details (ages, languages, locations, palettes) are hardcoded into agents or instructions.

The aesthetic baseline is editorial children's publishing: flat geometric vector illustration, retro travel-poster influence, calm structured layouts, light cultural depth. **Avoid:** watercolor, kawaii, clipart, AI-looking imagery, dense workbook formatting.

---

## 2. The Scrapbook Spec

Every book is described by a structured spec. The main agent fills this in from the brief + clarifying questions before any sub-agent fires.

| Field | Description |
|---|---|
| `book_slug` | Short kebab-case ID, e.g. `hungary-2026` |
| `audience` | List of children: age, reading/writing level, languages spoken, interaction modes (sticker, circle, draw, write, glue) |
| `language` | Book content language(s); monolingual or bilingual layout rules |
| `locations` | Ordered list of regions/cities/topics; one becomes the primary section |
| `page_count` | Target total (e.g. 50–60) and rough split per section |
| `format` | Page size, orientation, binding, finish, print target (home/professional) |
| `style` | Illustration style, palette directions per section, typography stance, density, references to emulate, references to avoid |
| `activities` | Recurring activity systems in scope (observation, ratings, guided drawing, glue-in, reflection, scavenger hunts, conversation prompts, map tracking) |
| `cultural_depth` | Light-touch / moderate / heavy. Default: light-touch and integrated |
| `production` | Print constraints: ink-consciousness, marker/pencil friendliness, background fill rules |
| `extras` | Sticker sheets, fold-outs, pockets, cover, colophon |
| `deadline` | Target use date and lead time for printing |

The spec lives at `{book_slug}/spec.md`. Lock it before production starts.

---

## 3. Pipeline

Linear with explicit human checkpoints. Do not skip a checkpoint to save time.

```
Brief intake
  ↓
Clarifying questions → Spec draft → [HUMAN: spec lock]
  ↓
Research (Researcher + Visual Scout in parallel) → [HUMAN: research review]
  ↓
Editorial plan: page-by-page outline (Editor) → [HUMAN: plan lock]
  ↓
Copy drafts (Copywriter) ⊕ Illustration prompts + generation (Illustrator) ⊕ Layout specs (Designer)
  ↓
Editor pass → revisions → [HUMAN: content lock]
  ↓
Affinity assembly (Affinity Producer)
  ↓
Final vision pass (Editor + Visual Scout) → [HUMAN: ship/no-ship]
  ↓
Export → handoff
```

Run the three middle tracks (copy, illustration, layout) in parallel where possible — they coordinate through the editorial plan, not through each other.

---

## 4. Brief Intake & Clarifying Questions

When a brief arrives:

1. Read it fully. Read any attached imagery using vision (the Read tool on image paths).
2. Draft the spec from the brief. For each field, mark confidence:
   - **≥ 0.85** — fill it in, flag with `⚠️ confirm`.
   - **< 0.85** — leave blank and ask.
3. Ask **only** the missing/ambiguous questions, batched. Do not ask one at a time. Do not ask what the brief already answers.
4. Common questions worth asking when not covered: deadline, print method, sticker scope, cover treatment, output format (`.afpub` source + PDF, PDF only, or other), budget for illustration generation.
5. Present the filled spec for lock. Do not start production until the user confirms.

---

## 5. Sub-Agent Roster

Seven specialised agents. Each does one thing. The Editor is the conductor.

Sub-agent definition files live in `.claude/agents/` (Anthropic format: frontmatter with `name`, `description`, `tools`, `model`, then prompt body). Briefs MUST NOT bake brief-specific details into agent files — pass those through the spec.

### 5.1 Editor (conductor)
- **Owns:** narrative flow, structure, age-appropriateness, emotional resonance, final QA.
- **Inputs:** spec, research outputs, drafts from copy/illustration/design.
- **Outputs:** editorial plan (page-by-page), feedback to other agents, final approval.
- **Tools:** Read, Write, Edit. No external tools — directs others.
- **Hard rule:** does not write first-draft copy, does not generate visuals.

### 5.2 Researcher
- **Owns:** facts, cultural notes, location detail, age-appropriate trivia, source list.
- **Inputs:** spec (locations, cultural_depth, language).
- **Outputs:** `research/{topic}.md` per location/topic with vetted facts, child-friendly framings, source links, and red-flag notes (anything inappropriate to surface near children).
- **Tools:** WebSearch, WebFetch, mcp__context7 (for any framework/tool questions), Read, Write.
- **Hard rule:** cite sources. No invented facts. Flag anything dark or politically charged for Editor decision rather than auto-including.

### 5.3 Visual Scout
- **Owns:** building the visual reference set; validating style direction against references.
- **Inputs:** spec (style), inspirational imagery from the brief folder.
- **Outputs:** `research/visual-references.md` — palette extractions, composition patterns, illustration-style traits to emulate, traits to avoid; per-section mood notes.
- **Tools:** Read (vision on images), WebSearch, WebFetch, mcp__chrome_devtools__take_screenshot (for capturing references from the live web when needed).
- **Hard rule:** every claim about the target style must point at a specific reference image or source.

### 5.4 Copywriter
- **Owns:** all on-page text — titles, captions, prompts, micro-stories, sticker labels.
- **Inputs:** approved editorial plan, spec (language, audience), research notes.
- **Outputs:** `drafts/copy/{section}.md` with per-page text blocks, multiple options where useful.
- **Tools:** Read, Write, Edit.
- **Hard rule:** writes in the spec's language(s) only. Tone matches spec. Keeps text short — pages must remain writable/usable. Provides alt versions on request, no defensive hedging.

### 5.5 Illustrator
- **Owns:** illustration prompt design and generation via Recraft API; per-image style consistency.
- **Inputs:** approved editorial plan, Visual Scout reference set, designer's per-page slot specs.
- **Outputs:** `assets/illustrations/{slug}.png` + `assets/illustrations/manifest.md` (one row per asset: page, slot, prompt, seed, version, status).
- **Tools:** Bash (calls Recraft via `curl`; API key from env), Read (vision QA on outputs), Write, Edit.
- **Hard rule:** never burn generation credits before Visual Scout + Designer sign off on style direction. Always vision-QA outputs before declaring done. Reject and re-prompt anything that looks AI-generic, watercolour, kawaii, or off-palette.

### 5.6 Designer
- **Owns:** page layouts, grid, typography decisions, colour blocking, illustration slot specs, per-page composition.
- **Inputs:** approved editorial plan, Visual Scout reference set, copy drafts.
- **Outputs:** `drafts/layouts/{section}.md` — per-page layout spec (grid, regions, type sizes, colour blocks, illustration slots with size + prompt brief, interactive zones).
- **Tools:** Read, Write, Edit, mcp__affinity__* (for prototyping layouts via the affinity-design skill), mcp__pencil__* (if .pen files become part of the workflow).
- **Hard rule:** respects margins, bleed, ink-consciousness, marker/pencil friendliness. Never alters approved copy.

### 5.7 Affinity Producer
- **Owns:** assembling the final book in Affinity Publisher from approved copy, illustrations, and layouts.
- **Inputs:** content lock from Editor (copy + illustrations + layout specs).
- **Outputs:** Affinity document path(s) on the user's Desktop, exported PDFs, status notes in `production/affinity.md` (document path, page index, asset map).
- **Tools:** mcp__affinity__add_sdk_hint, mcp__affinity__execute_script, mcp__affinity__list_library_scripts, mcp__affinity__list_sdk_documentation, mcp__affinity__read_library_script, mcp__affinity__read_sdk_documentation_topic, mcp__affinity__render_selection, mcp__affinity__render_spread, mcp__affinity__report_sdk_issue, mcp__affinity__save_script_to_library, mcp__affinity__search_sdk_hints, Skill, Read, Write.
- **Hard rule:** never saves a script to the library or reports an SDK issue without user sign-off; final export only after Editor + human sign-off; logs every Affinity script action with timestamp + design path.

---

## 6. Tools

| Purpose | Tool |
|---|---|
| Web research, facts | WebSearch, WebFetch |
| Library/API docs | mcp__context7 |
| Image understanding | Read (vision) on local image paths |
| Live-page reference capture | mcp__chrome_devtools__* |
| Affinity scripting & design assembly | mcp__affinity__* (see affinity-design skill) |
| Pencil .pen design files | mcp__pencil__* (if used) |
| Recraft illustration generation, edit, vectorize, custom-style training | `.claude/commands/recraft-imagegen/` skill (Python scripts; reads `RECRAFT_API_KEY` from `.env`) |
| File I/O | Read, Write, Edit |

The recraft-imagegen skill ships six scripts: `recraft_generate.py` (text-to-image), `recraft_edit.py` (img2img / inpaint / replace-bg / variate), `recraft_process.py` (vectorize / remove-bg / upscale), `recraft_style.py` (train a custom style from 1–5 reference images, returns a style_id), `recraft_explore.py` (divergent exploration), `recraft_account.py` (credit balance). See `.claude/commands/recraft-imagegen/SKILL.md` for full reference.

---

## 7. File Layout

Per-book artifacts live in their own subfolder:

```
P8-activity-book/
├── CLAUDE.md                     ← this file
├── .claude/
│   └── agents/                   ← seven sub-agent definitions
└── books/
    └── {book_slug}/
        ├── brief.md              ← original creative brief
        ├── spec.md               ← locked production spec
        ├── research/
        │   ├── {topic}.md
        │   └── visual-references.md
        ├── drafts/
        │   ├── editorial-plan.md
        │   ├── copy/{section}.md
        │   └── layouts/{section}.md
        ├── assets/
        │   └── illustrations/
        │       ├── {slug}.png
        │       └── manifest.md
        ├── production/
        │   └── affinity.md       ← document path, page index, asset map, export log
        └── exports/
            └── {book_slug}-vN.pdf
```

Books live under `books/`. The active Hungary book is at `books/hungary-2026/`. The public demo (Sunshine Coast) is at `books/sunshine-coast-demo/`.

---

## 8. Quality Bar & Defaults

These are the system's defaults. A spec can override any of them.

- **Illustration style:** flat geometric vector, retro travel-poster influence, simplified landmarks, limited palette, clean silhouettes.
- **Layout density:** low–medium. One main activity per page, two small max.
- **Cultural depth:** light-touch, embedded in activities, never textbook.
- **Multi-age accessibility:** activities support different depths of interaction without explicit "easy/hard" labelling.
- **Print readiness:** mostly cream/light backgrounds, ink-conscious, marker/pencil friendly.
- **Modularity:** location/topic sections share a colour-coded identity system (own colour, own icon language) so future sections can be added without redesign.
- **Tone:** exploratory, not instructional. Conversational. Child-friendly without talking down.

---

## 9. Hard Rules

- Do not start production before the spec is locked by the user.
- Do not generate illustrations before Visual Scout + Designer sign off on style direction AND a custom Recraft style has been trained from approved reference images.
- Every illustration in a book uses the same trained `style_id` for visual coherence.
- Do not assemble in Affinity before content lock.
- Do not export the final book or save scripts to the Affinity library without human sign-off.
- Do not invent cultural facts. Cite or omit.
- Do not bake brief-specific details (ages, language, locations, palette) into agent definition files. Those flow through the spec.
- Do not skip the vision QA pass on generated illustrations.
- Do not delete prior versions — version exports as `v1`, `v2`, etc.

---

## 10. Logging

For every book, maintain a one-file production log at `{book_slug}/log.md`. Each significant agent action gets one line: timestamp, agent, action, artifact link. Useful for debugging the pipeline and for postmortems on what went well/badly.

---
> Source: [sandra-arato/coldpress](https://github.com/sandra-arato/coldpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
