## ppt-as-code

> >


# PPT as Code

> Plan and build HTML-based presentations with a creator-first staged workflow.

**Core Pipeline**: `Ingest -> Normalize Source Material When Needed -> Derive Source Scenes When Needed -> Detect Input Mode -> If DSL: Parse deck.md -> Compile deck_source.json -> Route Mode -> Load References -> Diagnose Gaps -> Produce Artifacts -> Confirm Key Decisions -> Plan Visualizations -> Plan Images -> Run Pre-HTML QA -> Deliver HTML -> Optional Workbench Sync -> Optional PPTX Export`

---

## Mandatory Rules

> [!CAUTION]
> ### Serial Execution & Gate Discipline
>
> This workflow is a strict serial pipeline.
>
> 1. **SERIAL EXECUTION**: Steps must execute in order; output of each step becomes input for the next.
> 2. **BLOCKING = HARD STOP**: A `BLOCKING` step requires explicit user confirmation before continuing.
> 3. **NO SPECULATIVE EXECUTION**: Do not pre-build future artifacts before the current gate is cleared.
> 4. **KEEP THE ROUTE PROPORTIONAL**: Do not escalate a lightweight deck into a heavier stack unless the request clearly needs it.
> 5. **DEFAULT TO SHIPPING**: Deliver usable artifacts, prompts, or HTML handoff routes, not theory essays.
> 6. **FINAL ARTIFACT MUST RUN**: If the user wants a direct handoff, prefer a self-contained file or a locally runnable folder over a CDN-only prototype.

> [!IMPORTANT]
> ### Strict Execution Semantics For `basic` And `advanced`
>
> - Treat `basic` and `advanced` as strict finite-state workflows, not loose checklists.
> - Advance only one stage at a time. Do not silently bundle multiple blocking stages into one jump.
> - Silence, implied urgency, or generic requests such as "continue" do not count as permission to bypass required confirmations.
> - Only bypass a blocking step when the user explicitly names that override in a clear instruction, for example:
>   - `skip breakdown confirmation`
>   - `do not ask for step-by-step confirmation`
>   - `run the rest end-to-end without checkpoints`
> - If the user gives a partial override, skip only the named checkpoint and keep the rest of the workflow strict.
> - If the user does not clearly opt out, stay in the default strict step-by-step mode.

> [!IMPORTANT]
> ### Creator-First Delivery
>
> - Default delivery is **staged artifacts + HTML route or prompt pack + image plan**, not a long code dump.
> - When the user is a creator or wants to move fast, prefer staged prompts and intermediate artifacts over large code blocks by default.
> - Use full implementation only when the user explicitly asks for code or the workflow has reached the final HTML stage.

> [!IMPORTANT]
> ### PPTX Export Handoff
>
> - `ppt-as-code` remains an **HTML-first** presentation skill.
> - PPTX export is an optional **final-delivery post-process**, not a replacement for the HTML workflow.
> - Default export target is `html`; allow `pptx` or `both` when the user explicitly wants PowerPoint delivery.
> - When the target includes PPTX, finish the static HTML pass first, then produce `deck_manifest.json`, then hand off to `pptx-export-for-ppt-as-code`.
> - `deck_manifest.json` is the export bridge and source of truth for PPTX delivery; do not rely on ad-hoc DOM scraping as the primary route.
> - PPTX export is static-only: motion is downgraded to a static state, simple pages should stay editable, and complex pages may fall back to full-slide raster images.

> [!IMPORTANT]
> ### Slidev-Inspired DSL Input
>
> - `ppt-as-code` may accept an optional `deck.md` draft as a Slidev-inspired authoring input.
> - This is a **draft input layer**, not a compatibility promise for real Slidev projects or syntax.
> - When `deck.md` is present, parse it into `deck_source.json` before entering the normal `quick`, `basic`, or `advanced` workflow.
> - `deck.md` can speed up ideation, but it must not bypass `basic` or `advanced` confirmation gates.
> - `deck_source.json` is the normalized internal representation for DSL input; it is not the final delivery artifact.

> [!IMPORTANT]
> ### Source Normalization Rules
>
> - If the user starts from external source material rather than a slide outline, normalize that material before deck breakdown or script work.
> - When these adapters are available in the environment, use them proportionally:
>   - PDF -> `pdf_to_md.py`
>   - DOCX / EPUB / HTML / LaTeX -> `doc_to_md.py`
>   - ordinary web pages -> `web_to_md.py`
>   - high-friction pages such as WeChat or anti-bot pages -> `web_to_md.cjs`
> - The normalized markdown becomes the working source for deck planning; it is not the final deck.
> - If no adapter is available, do not block the workflow. Summarize the source manually and continue with an explicit note about the fallback.

> [!IMPORTANT]
> ### Source-To-Scenes Rules
>
> - If the starting material is a long article, PDF, document, or normalized web source, derive a preliminary scene map before writing the confirmed breakdown.
> - The source-to-scenes pass should cluster the material into likely slide groups, not final slides.
> - Use it to identify likely page sequence, candidate scene roles, strong quotes, strong stats, and image-worthy sections.
> - Treat `source_scene_map.md` as a planning accelerator, not as a replacement for the confirmed breakdown.

> [!IMPORTANT]
> ### Persistence Strategy
>
> - Default to **conversation-first artifacts**.
> - Only write files when the user explicitly wants persisted deck materials, or when the repo already has a clearly compatible project structure that invites file-based output.
> - When persistence is enabled, use conventional artifact names such as `deck_brief.md`, `theme_breakdown.md`, `style_options.md`, `deck_script.md`, `style_system.json`, `image_plan.md`, `index.html`, and `assets/`.

> [!IMPORTANT]
> ### PPT-First Grammar
>
> - `PPT-like` means stage-like: one active slide at a time, clear page state, and visible presentation furniture.
> - Do not let a deck drift into a long webpage, editorial article, or dashboard shell when the user explicitly asked for a presentation.
> - A finished deck must preserve clear slide roles such as cover, hook, concept, compare, quote, data, divider, and conclusion when relevant.

> [!IMPORTANT]
> ### Copy Relevance And Text-Density Rules
>
> - Every visible sentence on a slide must help the page thesis. If a line does not strengthen the point, cut it.
> - Do not add decorative filler such as generic scene-setting, vague inspiration lines, empty transition copy, or "PPT-sounding" meta language.
> - Avoid generic labels like `Overview`, `Background`, `Key Takeaway`, `Page 01`, `Closing Thoughts`, or similar scaffolding unless they carry real structural meaning for the audience.
> - Prefer one strong statement plus a small amount of supporting text over dense explanatory copy.
> - Do not solve crowded slides by shrinking type. If a page needs too much text, split the content into more slides or remove weak copy.
> - Avoid tiny captions, tiny footnotes, and small-print annotations unless they are required for sources, numbers, or compliance.
> - If a slide is image-led or chart-led, let the visual do the work. Do not add extra text just to fill space.
> - Use short, high-signal headlines. A slide title should carry a point, not just a topic label.

> [!IMPORTANT]
> ### Style Lock Rules
>
> - `quick` and `basic` still need a real visual direction; do not return a bare technical skeleton.
> - If style is undecided, recommend **3 to 4 design directions** before starting final HTML work.
> - Do not lock the final visual direction before the reference-image step when the chosen mode requires reference selection and browsing is available.
> - Once a reference is chosen, or the workflow has explicitly fallen back to style-word synthesis, translate the direction into structured design constraints before final implementation.

> [!IMPORTANT]
> ### Visualization Planning Rules
>
> - Charts, diagrams, infographics, and KPI cards are part of the deck structure, not late implementation garnish.
> - After the script is confirmed, decide page by page whether the page should be text-led, image-led, chart-led, diagram-led, or card-led.
> - Record those decisions in `visual_plan.md` before final HTML work begins.
> - Visualization choice must follow page thesis, not novelty. If a chart or diagram does not make the page clearer, do not force one.
> - Visualization placement must also be decided early: `full-width`, `left`, `right`, `top-band`, `bottom-band`, `center-focus`, or `split-2col`.
> - `quick` is intentionally excluded from the full visualization layer. Do not add chart-led, diagram-led, or card-led pages in `quick` unless the skill is explicitly upgraded to `basic` or `advanced`.

> [!IMPORTANT]
> ### Visualization Style Inheritance
>
> - Every chart or diagram must inherit the deck's design constraints.
> - Use the deck's typography, color logic, spacing rhythm, and emphasis language rather than introducing a separate visual system for charts.
> - Chart containers, labels, gridlines, fills, outlines, and cards must feel native to the deck, not like embedded third-party widgets.
> - When visual tokens are needed, derive them from the active design constraints instead of inventing them ad hoc.

> [!IMPORTANT]
> ### Image Workflow Rules
>
> - Image quality beats image quantity. A weak image is worse than no image.
> - Never search body images from the full deck topic alone.
> - For each page that needs an image, first compress the page into one thesis, then derive page-level keywords, then search.
> - If image download fails, keep the source link, record the failure, and hand the link to the user for manual download instead of blocking the run.

> [!IMPORTANT]
> ### Pre-HTML Quality Check
>
> - Before generating static HTML, run a lightweight QA pass over the approved artifacts.
> - The QA pass must check page sequence coherence, title hierarchy consistency, image-bearing slides that still lack usable assets or fallback links, and whether each slide has a clear page thesis.
> - If the QA pass finds gaps, resolve or surface them before HTML is treated as ready.

> [!IMPORTANT]
> ### Network And Tool Fallbacks
>
> - If browsing or downloading is unavailable, do not block the workflow.
> - In `advanced`, if web reference search is unavailable, skip the web-reference branch and derive structured design constraints directly from the chosen style direction and any user-provided inspiration.
> - If file persistence is unavailable or undesired, keep the same staged artifacts in conversation instead of forcing repo writes.

> [!IMPORTANT]
> ### Runtime Source Of Truth
>
> - Runtime behavior must come from `SKILL.md` and the active mode/reference files.
> - Stable feedback should be integrated into the main docs, not left in logs.

> [!IMPORTANT]
> ### Workbench Architecture Rules
>
> - A future visual workbench must not edit planning artifacts independently or let them drift.
> - Use `deck_model.json` as the unified workbench editing source when a canvas editor is in scope.
> - Workbench edits should first update `deck_model.json`, then synchronize upstream artifacts, then refresh HTML.
> - Treat the workbench as an internal authoring layer for `ppt-as-code` decks, not as a generic webpage editor.
> - The v1 workbench scope is limited to decks that already follow this skill's own artifact contracts.

> [!IMPORTANT]
> ### Workbench Sync Rules
>
> - `theme_breakdown.md`, `deck_script.md`, `style_system.json`, `visual_plan.md`, `image_plan.md`, and `deck_manifest.json` should not synchronize directly with one another.
> - `deck_model.json` is the bridge between the visual canvas and the artifact layer.
> - If a free-canvas operation cannot be projected back into the artifact system safely, prefer sync safety over visual freedom.
> - In those cases, require one of these outcomes:
>   - restrict the action
>   - mark the result as `needs_review`
>   - mark the result as `html_only_override`
> - Never silently create multiple competing sources of truth.

> [!IMPORTANT]
> ### Change Routing Rules
>
> - When the user asks to modify an existing deck, classify the request before editing anything.
> - Route the change to the primary upstream artifact first:
>   - structure change -> `theme_breakdown.md`
>   - copy change -> `deck_script.md`
>   - visualization change -> `visual_plan.md`
>   - image change -> `image_plan.md`
>   - style change -> `style_system.json`
>   - export behavior change -> `deck_manifest.json`
> - Global style changes such as font size, color system, spacing, radius, progress bar style, or chart label sizing belong in `style_system.json`.
> - Slide-specific style exceptions also belong in `style_system.json`, using a `slide_overrides` block instead of editing HTML first.
> - Do not treat `deck_manifest.json` as a content-editing source. It is a delivery-layer artifact.
> - In workbench mode, direct drag-and-resize edits should write to `deck_model.json` first, then sync into the correct artifact targets.
> - Do not edit `index.html` first unless:
>   - the user explicitly asks for an HTML-only hotfix
>   - the issue is implementation-specific
>   - no usable upstream artifact exists
> - After updating the primary artifact, regenerate only the affected downstream artifacts and then refresh HTML.

### Role Dispatch Protocol

This skill operates as a single inline agent - no role switching required.

---

## Resource Manifest

### UI Metadata

| File | Path | Purpose |
|------|------|---------|
| skill interface metadata | `${SKILL_DIR}/agents/openai.yaml` | display name, short description, and default prompt |

### References

| Resource | Path | Runtime Use |
|----------|------|-------------|
| quick mode workflow | `${SKILL_DIR}/references/quick-mode.md` | required only when mode = `quick` |
| basic mode workflow | `${SKILL_DIR}/references/basic-mode.md` | required only when mode = `basic` |
| advanced mode workflow | `${SKILL_DIR}/references/advanced-mode.md` | required only when mode = `advanced` |
| visual and image workflow | `${SKILL_DIR}/references/visual-and-images.md` | required when style, references, or images are in scope |
| visualization layer guide | `${SKILL_DIR}/references/visualization-layer.md` | required when charts, diagrams, infographics, or KPI cards are in scope |
| visualization engines guide | `${SKILL_DIR}/references/visualization-engines.md` | required when selecting chart or diagram engines |
| reference search pack | `${SKILL_DIR}/references/reference-search-pack.md` | required when browsing is used for reference locking or image/source discovery |
| component library guidance | `${SKILL_DIR}/references/component-libraries.md` | required only when the route needs libraries |
| PPTX export handoff | `${SKILL_DIR}/references/pptx-export-handoff.md` | required only when export target = `pptx` or `both` |
| source normalization guide | `${SKILL_DIR}/references/source-normalization.md` | required only when the input starts from PDF, DOCX, EPUB, HTML, LaTeX, or web pages |
| source-to-scenes guide | `${SKILL_DIR}/references/source-to-scenes.md` | required only when long source material needs pre-breakdown scene mapping |
| quality checker guide | `${SKILL_DIR}/references/quality-checker.md` | required before static HTML generation in non-trivial runs |
| deck DSL reference | `${SKILL_DIR}/references/deck-dsl.md` | required only when input mode = `dsl` |
| deck source contract | `${SKILL_DIR}/references/deck-source-contract.md` | required only when input mode = `dsl` |
| change routing guide | `${SKILL_DIR}/references/change-routing.md` | required when refining or modifying an existing deck |
| style system contract | `${SKILL_DIR}/references/style-system-contract.md` | required when style needs to be locked, edited, or regenerated |
| deck model contract | `${SKILL_DIR}/references/deck-model-contract.md` | required when the workbench or canvas editor is in scope |
| workbench architecture | `${SKILL_DIR}/references/workbench-architecture.md` | required when evaluating or building a visual workbench |
| workbench sync guide | `${SKILL_DIR}/references/workbench-sync.md` | required when workbench edits must flow back into artifacts |

### Workflows

| Workflow | Path | Purpose |
|----------|------|---------|
| `mode-delivery` | `${SKILL_DIR}/workflows/mode-delivery.md` | mode routing and output shaping |

---

## Workflow

### Step 1: Input Ingestion

`GATE`: The request is about a web-based presentation, HTML deck, slide system, reveal.js deck, presentation page, or a closely related deck-building workflow.

`EXECUTION`:

1. Extract the request shape:
   - topic
   - audience
   - delivery context: live presentation, shared link, async reading, or screen recording
   - source material form: notes, article, PDF, DOCX, EPUB, HTML, LaTeX, ordinary web page, high-friction web page, or `deck.md`
   - input mode: `structured`, `dsl`, or unspecified
   - export target: `html`, `pptx`, `both`, or unspecified
   - current stack: native HTML, React, Next.js, reveal.js, or unspecified
   - what the user already provided: notes, outline, raw material, references, images, existing code, or `deck.md`
2. Diagnose missing inputs:
   - theme breakdown
   - style direction
   - audience positioning
   - deck structure
   - visual references
   - image assets
3. Identify whether the request is about:
   - building from scratch
   - refining an existing deck
   - choosing a route
   - improving script, style, imagery, or motion
   - evaluating or defining a workbench / canvas editor
4. If the request is a modification, classify the primary change type:
   - `structure_change`
   - `copy_change`
   - `visualization_change`
   - `image_change`
   - `style_change`
   - `export_change`
   - `implementation_hotfix`
5. Decide the artifact persistence strategy:
   - default to conversation-first artifacts
   - enable file persistence only if the user asks for it or the repo clearly supports it
6. If persistence is enabled, choose the nearest reasonable project folder or deck folder that matches repo conventions.
7. Define the canonical artifact set for non-trivial runs:
   - `deck_brief.md`
   - `theme_breakdown.md`
   - `style_options.md`
   - `deck_script.md`
   - `style_system.json`
   - `visual_plan.md`
   - `image_plan.md`
   - `index.html`
   - `assets/`
8. If the export target includes PPTX, add the optional export artifacts:
   - `deck_manifest.json`
   - `output.pptx`
9. If the input mode is `dsl`, add the optional draft artifact:
   - `deck_source.json`
10. If the request starts from long source material, add the optional planning artifact:
   - `source_scene_map.md`
11. For non-trivial `basic` or `advanced` runs, add the optional QA artifact:
    - `qa_report.md`
12. If the request is about a workbench or visual editor, add the optional editor artifacts:
    - `deck_model.json`
    - `slide_overrides`

`CHECKPOINT`:

```markdown
## Step 1 Complete
- [x] Presentation request identified
- [x] Missing inputs diagnosed
- [x] Change type classified when relevant
- [x] Persistence strategy defined
- [ ] Next: auto-proceed to Step 1A
```

---

### Step 1A: Source Normalization, Source-To-Scenes, Input Mode Detection, And DSL Compilation

`GATE`: Step 1 complete; the request is clear enough to identify the input layer.

`EXECUTION`:

1. Detect whether source normalization is needed:
   - PDF -> use `pdf_to_md.py` when available
   - DOCX / EPUB / HTML / LaTeX -> use `doc_to_md.py` when available
   - ordinary web pages -> use `web_to_md.py` when available
   - high-friction web pages such as WeChat -> use `web_to_md.cjs` when available
2. If source normalization is needed:
   - read `${SKILL_DIR}/references/source-normalization.md`
   - normalize the material into markdown before deck planning begins
   - use the normalized markdown as deck-source input for later breakdown, script, and image work
   - if the adapter is unavailable, summarize manually and continue with an explicit fallback note
3. If the source is long-form and not already slide-shaped:
   - read `${SKILL_DIR}/references/source-to-scenes.md`
   - derive a scene map that clusters the source into likely page groups, scene roles, quotes, stats, and image-worthy sections
   - materialize it as `source_scene_map.md` when artifact persistence is enabled
4. Detect input mode:
   - `dsl` when the user provides `deck.md` or explicitly asks to start from a deck draft file
   - otherwise `structured`
5. If the input mode is `dsl`:
   - read `${SKILL_DIR}/references/deck-dsl.md`
   - read `${SKILL_DIR}/references/deck-source-contract.md`
   - parse `deck.md` as a Slidev-inspired DSL, not as real Slidev syntax
   - normalize the result into `deck_source.json`
6. Apply the required fallbacks while compiling:
   - missing frontmatter fields -> fill defaults
   - unknown layout -> downgrade to `bullets`
   - missing title -> infer from the first meaningful text
   - missing keywords -> leave empty for later image planning
   - unsupported Slidev-only syntax -> preserve as text and clearly label the boundary
7. Keep `deck.md` as a draft input only; do not treat it as the final source of truth for HTML or export.

`CHECKPOINT`:

```markdown
## Step 1A Complete
- [x] Source material normalized or fallback noted when needed
- [x] Source scenes derived when needed
- [x] Input mode detected
- [x] deck.md parsed when present
- [x] deck_source.json prepared when needed
- [ ] Next: auto-proceed to Step 2
```

---

### Step 2: Mode Routing

`GATE`: Step 1A complete; the request is clear enough to choose a mode.

`EXECUTION`:

1. Read `${SKILL_DIR}/workflows/mode-delivery.md`.
2. Respect an explicit user choice of `quick`, `basic`, or `advanced`.
3. If the user did not choose a mode, infer it:
   - choose `quick` for MVP decks, lightweight prototypes, and "get it running first"
   - choose `basic` for creator-facing deck planning that needs confirmed breakdown, confirmed script, and image handling before HTML
   - choose `advanced` for reference-driven deck design, richer style decisions, or a static-first then motion-follow-up workflow
4. Preserve the smallest route that satisfies the request.

`CHECKPOINT`:

```markdown
## Step 2 Complete
- [x] Execution mode chosen
- [x] Routing rationale prepared
- [ ] Next: auto-proceed to Step 3
```

---

### Step 3: Reference Loading

`GATE`: Step 2 complete; one execution mode has been chosen.

`EXECUTION`:

1. Load exactly one primary mode reference:
   - `quick` -> `${SKILL_DIR}/references/quick-mode.md`
   - `basic` -> `${SKILL_DIR}/references/basic-mode.md`
   - `advanced` -> `${SKILL_DIR}/references/advanced-mode.md`
2. Load `${SKILL_DIR}/references/visual-and-images.md` when style, references, or images are relevant.
3. Load `${SKILL_DIR}/references/visualization-layer.md` and `${SKILL_DIR}/references/visualization-engines.md` when charts, diagrams, infographics, or KPI cards may appear.
4. Load `${SKILL_DIR}/references/reference-search-pack.md` when browsing will be used for reference locking or image/source discovery.
5. Load `${SKILL_DIR}/references/component-libraries.md` only when the route truly needs component or chart libraries.
6. Load `${SKILL_DIR}/references/source-normalization.md` only when the source input needs normalization from document or web material.
7. Load `${SKILL_DIR}/references/source-to-scenes.md` only when long source material needs pre-breakdown scene mapping.
8. Load `${SKILL_DIR}/references/quality-checker.md` before static HTML generation in non-trivial runs.
9. Load `${SKILL_DIR}/references/pptx-export-handoff.md` only when the export target is `pptx` or `both`.
10. Load `${SKILL_DIR}/references/deck-dsl.md` and `${SKILL_DIR}/references/deck-source-contract.md` only when the input mode is `dsl`.
11. Load `${SKILL_DIR}/references/change-routing.md` when the request is about modifying an existing deck.
12. Load `${SKILL_DIR}/references/style-system-contract.md` when style needs to be locked, changed, or regenerated.
13. Load `${SKILL_DIR}/references/deck-model-contract.md`, `${SKILL_DIR}/references/workbench-architecture.md`, and `${SKILL_DIR}/references/workbench-sync.md` when a visual workbench or canvas editor is in scope.

`CHECKPOINT`:

```markdown
## Step 3 Complete
- [x] Primary mode reference loaded
- [x] Supporting references loaded only if relevant
- [ ] Next: auto-proceed to Step 4
```

---

### Step 4: Artifact Setup And Gap Diagnosis

`GATE`: Step 3 complete; the active mode workflow is loaded.

`EXECUTION`:

1. Define the artifact set for the chosen mode.
2. For `basic` and `advanced`, prepare the brief artifact first.
   - If persistence is enabled, materialize it as `deck_brief.md`.
   - Otherwise, keep it inline in the conversation.
3. Decide which artifacts are required now versus later:
    - `quick`: brief or outline, style directions, minimal HTML route
    - `basic`: brief, breakdown, style options, script, style system, visual plan, image plan, HTML
    - `advanced`: same as `basic`, plus structured design constraints and optional motion follow-up
    - long source input: add `source_scene_map.md` before the confirmed breakdown
    - `dsl`: add `deck_source.json` as the normalized draft input artifact
    - non-trivial runs: add `qa_report.md` as the pre-HTML QA artifact
    - `pptx` or `both`: add manifest and PPTX handoff after static HTML is ready
    - workbench scope: add `deck_model.json` as the unified canvas source
4. If the request is a modification, define the primary upstream artifact before touching HTML:
   - structure -> `theme_breakdown.md`
   - copy -> `deck_script.md`
   - visualization -> `visual_plan.md`
   - image -> `image_plan.md`
   - style -> `style_system.json`
   - export -> `deck_manifest.json`
   - implementation-only hotfix -> `index.html`
5. Record the missing gaps that must be resolved before HTML is allowed to begin.

`CHECKPOINT`:

```markdown
## Step 4 Complete
- [x] Artifact list defined for the chosen mode
- [x] Primary upstream edit target chosen when relevant
- [x] Immediate gaps recorded
- [ ] Next: branch into the chosen mode workflow
```

---

### Step 5: Quick Workflow

`GATE`: Chosen mode is `quick`.

`EXECUTION`:

1. Produce a lightweight deck brief:
   - topic
   - audience
   - core message
   - missing inputs
2. If the theme is still vague, create a lightweight outline before implementation.
3. If normalized source material exists, use a compact source-to-scenes pass to compress it into likely page groups before locking the quick outline.
4. If style is missing, recommend **3 to 4 style directions** and clearly mark one as the default recommendation.
5. Continue with the recommended direction unless the user explicitly wants to choose.
6. Run a lightweight pre-HTML QA pass before finalizing the quick HTML route.
7. Produce:
   - a minimal slide structure
   - one cover direction
   - a minimum viable HTML route or short prompt pack
8. If the user asked for images in `quick`, use the page-thesis keyword workflow from `visual-and-images.md`, but keep the image plan lightweight.
9. Do not add the visualization layer in `quick`. If the deck needs charts, diagrams, infographics, or KPI cards, upgrade to `basic` or `advanced`.
10. If the user wants `pptx` or `both`, keep the slide roles clear enough to support a later manifest handoff.
11. Include an upgrade path toward `basic`.

`CHECKPOINT`:

```markdown
## Step 5 Complete
- [x] Lightweight structure prepared
- [x] Visual direction prepared
- [x] Minimal HTML route or prompt pack prepared
- [ ] Next: auto-proceed to Step 17
```

---

### Step 6: Basic Workflow A - Breakdown First

`GATE`: Chosen mode is `basic`.

`EXECUTION`:

1. Prepare the brief artifact with current inputs and missing gaps.
2. Recommend **3 to 4 design directions** in the style-options artifact.
3. Always include arrow / progress / page-furniture suggestions as part of the style pack, using options such as:
   - floating
   - transparent
   - large footprint
   - small footprint
4. If normalized long-form source exists, derive `source_scene_map.md` before the confirmed breakdown.
5. Prepare the theme-breakdown artifact before any slide script exists.
6. The theme-breakdown artifact must define:
   - the deck thesis
   - the page sequence
   - what each slide is trying to say
   - which pages likely need images
7. If persistence is enabled, materialize these as `deck_brief.md`, `style_options.md`, `theme_breakdown.md`, and `source_scene_map.md` when relevant.

`BLOCKING`: Ask the user to confirm the theme breakdown before any script generation begins.

`CHECKPOINT`:

```markdown
## Step 6 Complete
- [x] Brief artifact prepared
- [x] Style options prepared
- [x] Theme breakdown prepared
- [ ] Next: BLOCKING - wait for breakdown confirmation
```

---

### Step 7: Basic Workflow B - Script Lock

`GATE`: Chosen mode is `basic`, and the user has confirmed the theme breakdown.

`EXECUTION`:

1. Lock the chosen style direction.
2. If local writing-style notes exist, scan likely style docs such as `voice_profile.md`, `brand.md`, `writing_style.md`, or project notes.
3. Translate the chosen style direction into `style_system.json` or an equivalent structured style object before final HTML work begins.
4. Prepare the deck-script artifact.
5. The deck-script artifact must define slide by slide:
   - scene or page purpose
   - headline or core copy
   - supporting copy or cue text
6. Apply the copy-relevance and text-density rules:
   - remove any line that does not directly support the page thesis
   - keep the visible copy sparse enough that the slide can stay large-type
   - if the page only works with dense small text, split it into multiple slides instead of compressing it
7. If persistence is enabled, materialize these as `style_system.json` and `deck_script.md`.
8. Do not start keyword extraction or image search yet.

`BLOCKING`: Ask the user to confirm the slide script before visualization and image work begin.

`CHECKPOINT`:

```markdown
## Step 7 Complete
- [x] Style locked
- [x] style_system.json prepared when relevant
- [x] Local voice guidance applied when available
- [x] Deck script prepared
- [ ] Next: BLOCKING - wait for script confirmation
```

---

### Step 8: Basic Workflow C - Visualization Planning

`GATE`: Chosen mode is `basic`, and the user has confirmed the slide script.

`EXECUTION`:

1. Read `${SKILL_DIR}/references/visualization-layer.md`.
2. Read `${SKILL_DIR}/references/visualization-engines.md`.
3. Prepare the `visual_plan.md` artifact or inline equivalent.
4. For each slide, decide whether it is:
   - `text-led`
   - `image-led`
   - `chart-led`
   - `diagram-led`
   - `card-led`
5. If a slide is visualized, record:
   - chosen engine
   - placement
   - chart or diagram type
   - content source
   - style constraints inherited from the deck
   - fallback mode
6. Do not force visualization on a page that works better as text or image.
7. If persistence is enabled, materialize it as `visual_plan.md`.

`CHECKPOINT`:

```markdown
## Step 8 Complete
- [x] Visualization decisions prepared
- [x] visual_plan.md prepared when relevant
- [ ] Next: auto-proceed to Step 9
```

---

### Step 9: Basic Workflow D - Image Plan

`GATE`: Chosen mode is `basic`, and the user has confirmed the slide script.

`EXECUTION`:

1. Prepare the image-plan artifact.
2. For each slide that needs imagery:
   - compress the slide into one thesis
   - derive exactly **1 to 2 page-level keywords**
   - search using those keywords plus the chosen style direction when browsing is available
   - if browsing is unavailable, provide search strings and image intent instead of pretending search happened
3. Attempt to download selected images into `assets/` only when downloading is available and persistence is enabled.
4. If download fails, record in the image-plan artifact:
   - slide number
   - keywords
   - source URL
   - failure reason
   - note: `user needs to download manually`
5. If persistence is enabled, materialize the plan as `image_plan.md`.
6. Do not hide failed downloads. Surface the links to the user.

`BLOCKING`: Ask the user to confirm the script, visualization plan, and image plan before HTML implementation begins.

`CHECKPOINT`:

```markdown
## Step 9 Complete
- [x] Image plan prepared
- [x] Slide-level keywords extracted
- [x] Download attempts or fallback links recorded
- [ ] Next: BLOCKING - wait for script, visualization, and image confirmation
```

---

### Step 10: Basic Workflow E - Static HTML

`GATE`: Chosen mode is `basic`, and the user has confirmed the script, visualization plan, and image plan.

`EXECUTION`:

1. Read `${SKILL_DIR}/references/quality-checker.md`.
2. Run a lightweight QA pass over the approved breakdown, script, visual plan, and image plan.
3. Verify in `qa_report.md` or inline QA notes:
   - page sequence coherence
   - title hierarchy consistency
   - visualization type and placement fit
   - image gaps or fallback links
   - page-thesis coverage
4. Resolve or surface any remaining gaps before HTML generation.
5. Generate the static HTML output.
6. Keep the deck presentation-first:
   - one active slide at a time
   - clear navigation state
   - visible progress or page furniture
7. Do not reduce font sizes just to fit too much copy on one slide. Remove text or split the page instead.
8. If persistence is enabled, materialize the final output as `index.html` and `qa_report.md` when needed.
9. If the user wants a final artifact, prefer a self-contained HTML file or a locally runnable folder.

`CHECKPOINT`:

```markdown
## Step 10 Complete
- [x] Static HTML generated
- [x] Presentation grammar preserved
- [ ] Next: auto-proceed to Step 17
```

---

### Step 11: Advanced Workflow A - Direction Selection

`GATE`: Chosen mode is `advanced`.

`EXECUTION`:

1. Prepare the brief artifact.
2. Recommend **3 to 4 design directions**.
3. If browsing is available, search for direction-shaping inspiration when it helps the choice.
4. In the style-options artifact, explain each direction in terms of:
   - tone
   - hierarchy
   - palette mood
   - imagery
   - page furniture
   - arrow / progress styling
5. If persistence is enabled, materialize the brief and style artifacts as `deck_brief.md` and `style_options.md`.

`BLOCKING`: Ask the user to choose one direction before final visual locking begins.

`CHECKPOINT`:

```markdown
## Step 11 Complete
- [x] Brief artifact prepared
- [x] Style options prepared
- [ ] Next: BLOCKING - wait for direction choice
```

---

### Step 12: Advanced Workflow B - Reference Lock Or Fallback

`GATE`: Chosen mode is `advanced`, and the user has chosen a direction.

`EXECUTION`:

1. If browsing is available, search for **3 PPT or slide-design reference images** that match the chosen direction.
2. Present found references with:
   - link
   - short label
   - why it fits
   - what to borrow from it
3. If browsing is unavailable, do not block on web references.
4. In the no-browsing path, derive the visual lock directly from:
   - the chosen style direction
   - any user-provided inspiration
   - the deck topic and audience
5. Do not lock the final visual direction before this step completes.

`BLOCKING`: Ask the user to choose one reference image if web references were produced. If the workflow used the no-browsing fallback, skip this blocking step and proceed automatically.

`CHECKPOINT`:

```markdown
## Step 12 Complete
- [x] Reference branch or no-network fallback completed
- [x] Final visual not locked prematurely
- [ ] Next: proceed to Step 13
```

---

### Step 13: Advanced Workflow C - Script Lock

`GATE`: Chosen mode is `advanced`, and either a reference image has been selected or the no-browsing fallback has completed.

`EXECUTION`:

1. Translate the chosen reference, or the chosen style direction in fallback mode, into structured design constraints before HTML work begins.
2. Convert those constraints into `style_system.json` or an equivalent structured style object.
3. If normalized long-form source exists, derive `source_scene_map.md` before locking the final script shape.
4. If local writing-style notes exist, scan likely style docs such as `voice_profile.md`, `brand.md`, `writing_style.md`, or project notes.
5. Prepare the deck-script artifact.
6. Apply the copy-relevance and text-density rules:
   - remove any line that does not directly support the page thesis
   - keep the visible copy sparse enough that the slide can stay large-type
   - if the page only works with dense small text, split it into multiple slides instead of compressing it
7. If persistence is enabled, materialize these as `style_system.json`, `deck_script.md`, and `source_scene_map.md` when relevant.

`BLOCKING`: Ask the user to confirm the slide script before visualization and image planning begin.

`CHECKPOINT`:

```markdown
## Step 13 Complete
- [x] Structured design constraints prepared
- [x] style_system.json prepared when relevant
- [x] Deck script prepared
- [ ] Next: BLOCKING - wait for script confirmation
```

---

### Step 14: Advanced Workflow D - Visualization And Image Plan

`GATE`: Chosen mode is `advanced`, and the user has confirmed the slide script.

`EXECUTION`:

1. Read `${SKILL_DIR}/references/visualization-layer.md`.
2. Read `${SKILL_DIR}/references/visualization-engines.md`.
3. Prepare the `visual_plan.md` artifact or inline equivalent.
4. For each slide, decide whether it is:
   - `text-led`
   - `image-led`
   - `chart-led`
   - `diagram-led`
   - `card-led`
5. For each visualized slide, record:
   - chosen engine
   - placement
   - chart or diagram type
   - content source
   - style constraints inherited from the deck
   - fallback mode
6. For each slide that needs imagery:
   - compress the slide into one thesis
   - derive exactly **3 to 4 page-level keywords**
   - search using those keywords plus the chosen style direction when browsing is available
   - if browsing is unavailable, provide search strings and image intent instead of pretending search happened
7. Attempt to download images into `assets/` only when downloading is available and persistence is enabled.
8. If download fails, record the same fallback fields required by `basic` and surface the source link to the user.
9. If persistence is enabled, materialize these as `visual_plan.md`, `image_plan.md`, and `source_scene_map.md` when relevant.
10. If the export target includes PPTX, prepare the slide content with explicit export hints after the static deck is approved.

`BLOCKING`: Ask the user to confirm the script, visualization plan, and image plan before static HTML begins.

`CHECKPOINT`:

```markdown
## Step 14 Complete
- [x] visual_plan.md prepared when relevant
- [x] Image plan prepared
- [ ] Next: BLOCKING - wait for script, visualization, and image confirmation
```

---

### Step 15: Advanced Workflow E - Static HTML First

`GATE`: Chosen mode is `advanced`, and the user has confirmed the script, visualization plan, and image plan.

`EXECUTION`:

1. Read `${SKILL_DIR}/references/quality-checker.md`.
2. Run a lightweight QA pass over the approved breakdown, script, visual plan, image plan, and design constraints.
3. Verify in `qa_report.md` or inline QA notes:
   - page sequence coherence
   - title hierarchy consistency
   - visualization type and placement fit
   - image gaps or fallback links
   - page-thesis coverage
4. Resolve or surface any remaining gaps before HTML generation.
5. Generate the static HTML output first.
6. Keep the approved content order and visual hierarchy intact.
7. Do not add advanced motion in this step unless the user explicitly asked to skip the static-first pass.
8. Do not reduce font sizes just to fit too much copy on one slide. Remove text or split the page instead.
9. If persistence is enabled, materialize the output as `index.html` and `qa_report.md` when needed.
10. If the export target includes PPTX, treat this static pass as the structure lock for `deck_manifest.json`.

`BLOCKING`: Ask the user whether they want a motion pass after reviewing the static deck.

`CHECKPOINT`:

```markdown
## Step 15 Complete
- [x] Static HTML generated
- [x] Static-first delivery preserved
- [ ] Next: BLOCKING - wait for motion decision
```

---

### Step 16: Advanced Workflow F - Optional Motion Pass

`GATE`: Chosen mode is `advanced`, and the user explicitly asked for a motion pass after reviewing the static deck.

`EXECUTION`:

1. Add motion only after the static deck is approved.
2. Treat motion as pacing, not decoration.
3. Use `transform` and `opacity` by default.
4. Preserve:
   - one active slide at a time
   - keyboard navigation
   - `prefers-reduced-motion`
5. Do not rewrite the approved script to justify animation.

`CHECKPOINT`:

```markdown
## Step 16 Complete
- [x] Motion layer added after approval
- [x] Reduced-motion support preserved
- [ ] Next: auto-proceed to Step 17
```

---

### Step 17: Delivery

`GATE`: The chosen mode workflow has completed.

`EXECUTION`:

1. Deliver in this order:
    - export target
    - chosen mode and why
    - artifact status
    - `deck_model.json` status when a workbench route is in scope
    - visual direction or chosen reference or fallback direction
    - script status
    - image status, including failed downloads and manual-download links
    - static HTML status
   - `deck_manifest.json` status when the export target includes PPTX
   - `output.pptx` status or handoff status when the export target includes PPTX
   - optional motion status when relevant
2. If the export target includes `pptx` or `both`:
   - verify the static HTML pass is complete and approved enough to freeze slide structure
   - prepare `deck_manifest.json` with `deckTitle`, `slideSize`, `themeTokens`, and per-slide export hints
   - hand off `index.html`, `deck_manifest.json`, and `assets/` to `pptx-export-for-ppt-as-code`
3. Before calling the deck final, verify:
   - it still feels like a presentation
   - page furniture exists where needed
   - slide roles read clearly
   - images are page-fit, not just topic-related
   - the deck is self-contained enough for the requested handoff

`CHECKPOINT`:

```markdown
## Step 17 Complete
- [x] Final delivery follows the staged artifact order
- [x] Presentation-first checks performed
- [x] Image fallback links surfaced when needed
- [x] PPTX handoff prepared when requested
```

---

## Optional Workbench Route

Use this route only when the user is evaluating, specifying, or building a visual PPT-style editor for `ppt-as-code`.

- Treat the workbench as a new product layer above the existing artifact workflow.
- Use `deck_model.json` as the unified workbench editing source.
- Preserve these module boundaries:
  - Canvas for drag, resize, align, layer, and grouping interactions
  - Inspector for typography, color, spacing, shadow, radius, and chart-style controls
  - Outline for slide ordering, lock state, visibility, and element tree browsing
  - Sync Engine for projection between `deck_model.json` and the artifact layer
  - Preview / Export for HTML render validation and PPTX handoff preparation
- Preserve these V1 element types only:
  - `text`
  - `image`
  - `chart`
  - `diagram`
  - `card`
  - `shape`
  - `container`
- Prefer sync integrity over unlimited visual freedom.
- If a free-canvas action cannot project back into the artifact system safely, restrict it or mark it as `needs_review` or `html_only_override`.

---
> Source: [Russell-cell/PPT-as-code](https://github.com/Russell-cell/PPT-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
