## better-ppt-html-deck

> Create better-looking, professional, editable HTML/CSS/JS slide decks. Use when the user asks to make a better-looking PPT, presentation, deck, slide deck, PowerPoint-style HTML deck, visually polished report, course deck, pitch deck, social carousel, or when they provide a topic/material/reference images and want Codex to plan content, choose a visual direction, generate a mandatory style preview for approval, then build an editable HTML deck with preview, present, PDF/PNG/image-PPTX export.


# Better PPT HTML Deck

Use this skill to help users quickly create better-looking PPT-style decks rendered from HTML/CSS/JS. The goal is not to build a general PPT workbench, SaaS design tool, Figma clone, or Canva clone. Editing, image replacement, preview, presenting, and export exist only to support fast creation and revision of a more beautiful, more professional deck.

## Hard Rules

- Do not generate visual direction, image assets, or `style-preview.png` when the user's brief is ambiguous. First ask 10-20 Socratic clarification questions, wait for the user's answers, then proceed.
- Generate a style preview before building the final deck project.
- Do not build the final `deck-project/` until the user explicitly approves `style-preview.png` or clearly says to proceed with that style.
- Render the final deck from HTML/CSS/JS. HTML is the source of truth; PDF, PNG, and PPTX are exports.
- All final decks and exports must be 16:9 unless the user explicitly asks for another aspect ratio. Use a fixed 1600x900 (or proportional) render viewport for screenshots/PNG, 16in x 9in for PDF, and an explicit 13.333333in x 7.5in PPTX layout. Do not insert non-16:9 slide screenshots into PPTX pages or stretch individual images into mismatched frames.
- Every final generation must use a fresh project workspace/output directory. Do not overwrite or keep reusing an existing `deck-project/` for a new generation or regenerated deck. Use a unique slug/timestamp path, and ensure exports, screenshots, dev server, and localStorage identity all point to that new project.
- Do not generate whole slides as static AI images. Image generation is allowed only for local visual assets such as cover hero art, background textures, illustrations, product concepts, mood images, or supporting images.
- For final decks, generate bitmap supporting imagery by default unless the user explicitly refuses image generation or the deck is strictly data-only. Do not ship a polished deck with only SVG placeholders, CSS art, or empty image slots.
- Generated images must be default-loaded into the deck: copy them into `deck-project/public/assets/images`, reference those local files from `src/data/deck.json`, render them on the relevant slides through `EditableImage`, and log them in `image-prompts.md` plus `asset-source-log.md`.
- When the user asks for GPT Image, gpt-image-2, AI-generated visuals, richer bitmap artwork, or a style that benefits from texture/atmosphere, use image generation for suitable local assets instead of falling back to plain SVG placeholders.
- Prefer HTML/CSS/SVG for diagrams, architecture drawings, terminal windows, code blocks, data cards, timelines, comparison tables, UI frames, charts, and layout.
- Keep all primary slide text in `src/data/deck.json`; do not hardcode main content inside components.
- Add a unique `deck.projectId` and `deck.dataVersion` to `src/data/deck.json` for each final build. Autosave must use a versioned storage key derived from these fields, and `loadDeck()` must fall back to the bundled seed deck when stored data belongs to an older project/version. Never use a fixed localStorage key across regenerated decks.
- Make the final deck directly editable: every visible title, subtitle, list item, card label, diagram label, code/prompt snippet, quote, speaker note, and image/logo must be editable or replaceable from Edit Mode.
- Include browser-side one-click PPTX export. Do not make users run a terminal command for the primary PPT export path.
- Final projects must be run by Codex before delivery. Do not hand off a deck project with only "run these commands" instructions; install dependencies when needed, run build/validation/screenshot QA, start or verify a local preview/dev server when the project requires one, and give the user the working URL or directly openable file path.
- Include meaningful visual assets or image slots. A final deck with no replaceable images/visuals is incomplete unless the user explicitly requests text-only slides. For polished decks, include generated bitmap assets by default: at minimum one cover/hero bitmap plus 2-5 supporting bitmap images for an 8-12 slide deck, default-loaded into the slides.
- Default layouts must feel substantial and presentation-ready: avoid slides where content only occupies a small island, avoid repeated corner thumbnails, and make primary content plus primary visual occupy most of the slide.
- Build decks as designed page systems, not title-and-bullets pages. Every final deck should use a mix of large hero compositions, split narrative/image pages, KPI/data pages, timelines, comparison tables, quote/testimonial cards, dashboards, roadmap/process pages, and closing CTA pages as appropriate.
- Default content must be complete enough to present: do not under-fill decks with only short phrases. For pitch/report/product decks, every non-cover/non-section slide should include evidence, metrics, examples, cases, objections, roadmap detail, or decision criteria. For training, onboarding, learning, course, or workshop decks, include practice guidance and a closing upload/QR placeholder by default.
- If the user provides visual references, explicitly translate them into layout rules before building: page grid, headline scale, visual/data ratio, card treatment, chart/table density, background treatment, and image-generation targets. The final deck should visibly borrow the composition logic, not merely the colors.
- When the user asks for generated images or provides polished visual references, create enough local generated bitmap assets to make the deck feel designed. Do not leave image slots as generic placeholders in the final deck unless the user has explicitly declined image generation or the slot is intentionally a replaceable product/logo/QR placeholder.

## Standard Workflow

### 1. Understand The Request

First decide whether the brief is clear enough to plan. A brief is clear enough only when the following are known or safely inferable:

- Topic, purpose, audience, scenario, page count, aspect ratio, language.
- Whether it is a formal report, course, pitch, product launch, training deck, or social carousel.
- Content source, key message, desired outcome, content density, visual preferences, uploaded materials, uploaded reference images.
- Constraints such as deadline, must-include facts, brand rules, export needs, confidentiality, and whether AI-generated images are acceptable.
- Defaults: `16:9`, Chinese, 8-12 slides when unspecified, and the goal of clearer structure plus stronger visual impact.

If two or more important fields are missing, or the content/visual goal could plausibly branch into different decks, stop before visual generation and ask 10-20 Socratic clarification questions. The questions should help the user discover what they want, not merely fill a form.

Use this question shape:

- Start with goal and audience: why this deck exists, who must be persuaded/taught/aligned, what action should happen after the deck.
- Then content: source materials, non-negotiable facts, proof points, examples, data, outline, sections to include or avoid.
- Then narrative: desired angle, emotional tone, level of detail, what the audience already knows, likely objections.
- Then visual direction: reference images, brand rules, image-generation comfort, preferred/forbidden styles, diagram/chart needs.
- Then delivery/export: page count, language, aspect ratio, presenter vs self-reading, editable/export priorities, deadline.

Ask concise numbered questions, usually 10-12 for a moderately unclear request and up to 20 when the deck is high-stakes, brand-sensitive, data-heavy, or has no source material. After asking, wait for the user's answers. Do not create `outline.md`, `visual-direction.md`, generated images, or `style-preview.png` until the answers are received or the user explicitly says to proceed with reasonable assumptions.

Read `references/socratic-briefing.md` when clarification is needed.

Use Fast Mode for quick drafts when requested. Use Pro Mode by default for formal work.

### 2. Plan Content

Create `outline.md` with:

- Deck title and narrative logic.
- One core point per slide, plus enough supporting substance to present the point.
- Slide title, information hierarchy, recommended slide type, likely visual assets, generated bitmap assets, charts, diagrams, screenshots, code blocks, visual notes, and speaker-note intent.
- A per-slide layout/composition note such as `hero-split`, `wide-workflow`, `central-diagram`, `dashboard-grid`, `case-study`, `practice-upload`, or `closing-cta`.
- A per-slide content payload: 2-5 concrete support items, data cards, comparison rows, timeline milestones, quote/testimonial details, annotations, or speaker-note prompts. Avoid writing only the title and a generic subtitle.
- A per-slide visual asset plan: `generated-bitmap`, `editable-diagram`, `chart`, `table`, `mockup-frame`, `photo-frame`, `map`, `timeline`, or `none-intentional`, plus the asset prompt when generation is needed.
- A deck-level generated image plan: list the exact bitmap assets to generate, their slide usage, aspect ratio, prompt intent, file name, and where they will be loaded in `deck.json`.

Avoid dense small text and overloaded slides, but do not make empty slides. A good slide is spacious, not thin.

### 3. Define Visual Direction

If reference images are provided, analyze palette, layout, typography mood, component style, background style, image language, and visual emotion.

If no reference images are provided, infer a suitable visual direction from topic, audience, industry, and content tone. Browse or research only when current public facts or domain-specific visual conventions matter.

Create `visual-direction.md` with style name, rationale, colors, typography, layout rules, image style, component style, background style, and forbidden choices.

Read these references as needed:

- `references/design-principles.md`
- `references/visual-style-rules.md`
- `references/layout-content-rules.md`
- `references/reference-layout-systems.md`
- `references/style-preview-guide.md`
- `references/style-feedback-map.md`

### 4. Generate Style Preview

This step is mandatory. Create a style board, not the final deck:

- `styleframe.html`
- `styleframe.css`
- `style-preview.png`

Prefer code-rendered HTML/CSS for the preview. Use image generation only for local visual assets inside the board. If the target style needs a bitmap hero, mood image, texture, product concept, or illustration, generate a representative local asset for the style preview. The board should show a cover preview, content slide preview, diagram/chart slide preview, palette, type, component treatment, icon style, and style keywords.

After producing `style-preview.png`, stop and ask for approval or feedback. Do not continue into final deck generation.

State flow:

```txt
ambiguous_request -> socratic_questions_asked -> waiting_for_brief_answers -> brief_ready
draft -> style_preview_generated -> waiting_for_approval -> approved -> deck_building -> deck_ready
```

For revisions:

```txt
waiting_for_approval -> revision_requested -> style_preview_regenerated -> waiting_for_approval
```

Record preview revisions in `revision-log.md`:

```md
## v2 Revision Log
- User feedback:
- Revision direction:
- Keep:
- Remove:
- New visual strategy:
```

### 5. Lock Style And Content

After style approval, create `visual-lock.json`. Treat it as the visual contract for the final deck. Include style name, aspect ratio, colors, typography, layout tokens, component choices, image treatment, and visual rules.

After content direction is stable, create `content-lock.md` with deck goal, audience, final slide count, narrative, each slide's core point, type, title, summary, locked facts, layout/composition role, and areas where wording may be optimized.

For training, learning, onboarding, workshop, or method decks, lock a practice-oriented ending: at minimum a closing CTA slide with 2-3 next steps and an editable QR/upload/resource placeholder.

### 6. Build Final HTML Deck

Create the final `deck-project/` only after style approval, and create it inside a fresh unique workspace/output directory such as `outputs/<topic-slug>/<timestamp-or-version>/deck-project/`. If a previous project exists for the same topic, treat it as a reference only; do not overwrite it or reuse its `exports/`, browser storage key, dev server URL assumptions, or generated asset paths. Prefer a React + Vite project unless the user or repo context clearly favors another stack.

Required structure:

```txt
deck-project/
|- index.html
|- package.json
|- src/
|  |- App.tsx
|  |- main.tsx
|  |- data/deck.json
|  |- editor/
|  |- slides/
|  |- components/
|  |- theme/
|  |- utils/
|  `- styles/global.css
|- public/assets/
|- scripts/
|- exports/
`- README.md
```

The deck must support:

- Edit Mode: click-to-edit title/body/list/notes, click or drag image/logo replacement, autosave to localStorage or IndexedDB, import/export `deck.json`.
- Storage Isolation: autosave must use a project/version-specific key derived from `deck.projectId` and `deck.dataVersion`. On load, reject stale stored JSON when its project/version metadata does not match the bundled `deck.json`, so every browser shows the latest generated seed deck on first open.
- Preview Mode: hide editor controls and helpers; show near-export result.
- Present Mode: fullscreen, arrow keys, space for next, Esc exit, page number, progress, optional speaker notes.
- Export Mode: browser-side one-click image-based PPTX, PDF, per-slide PNG, `deck.json` import/export. The PPTX button must be visible in the app UI, not only documented as `npm run export:pptx`. Browser-side PPTX export must capture each complete slide as a fixed 16:9 image and place that image full-bleed on a 16:9 PPTX slide; do not rebuild slides from individual PPTX text/image objects for the default export path because that can stretch images and drift layout.
- Visual Assets: include at least one editable/replaceable image or SVG visual on the cover and enough supporting visuals across the deck to avoid a text-only result. Prefer GPT Image / built-in image generation for cover hero, mood, product, background, or illustration assets when it improves quality. For polished decks, generated bitmap assets are required by default: aim for the cover plus 35-60% of content slides, with the remaining slides using code-rendered charts, tables, diagrams, maps, mockups, or data dashboards. Register all visuals in `src/data/deck.json`, and make the generated images the default visible assets.
- Layout Scale: each slide must use a large-format composition. Do not place every visual in the same corner. Alternate visual placement and use large hero frames, split layouts, central diagrams, wide process flows, dashboards, timeline bands, comparison matrices, testimonial stacks, map/data compositions, pricing grids, or full-bleed background image treatments. If a visual is small, it must have a clear role such as QR, logo, icon, or annotation.
- Content Depth: each non-cover/non-section slide should include 2-5 meaningful supporting points, steps, criteria, examples, annotations, or data cards unless it is intentionally a `big-idea` or `quote` slide. At least half of the content slides in pitch/report decks should include structured proof such as numbers, named modules, feature rows, case snippets, roadmap phases, competitive criteria, or financial/market assumptions.
- Reference-Led Richness: when reference images show dense professional pages, emulate that richness with editable HTML/CSS components: KPI cards, line/bar charts, progress rings, roadmaps, comparison grids, award lists, pricing cards, team/testimonial cards, maps, phone/laptop mockups, and annotated image frames. Do not flatten these into plain bullet lists.
- Practice Upload Ending: for learning/course/workshop decks, include a closing template with an editable QR/image placeholder and text such as `scan to upload practice`, `submit assignment`, `upload screenshot`, or `practice entry`.

Before considering the deck complete, perform an editability audit:

- Click-to-edit works for slide title, subtitle/body, list items, card headings, card bodies, diagram node labels, code/prompt text, quote text, closing CTA, and speaker notes.
- Image/logo/visual slots support click upload, drag upload, JPG/PNG/WebP/SVG, object-fit cover/contain where relevant, and local autosave.
- All edited values flow back to `deck.json` state and can be exported/imported.
- Browser autosave cannot mask regenerated content: bump `deck.dataVersion` whenever generated content/assets change, and verify in a clean browser context or with storage cleared/version-mismatched that generated images appear on first load.
- Generated bitmap assets are copied into `public/assets/images`, logged in `image-prompts.md` and `asset-source-log.md`, and never referenced from the generation tool's default output folder.
- Every generated bitmap asset is actually used by at least one slide through `deck.json`. A generated image that exists only in a log or temp folder does not count.
- Closing upload/QR/resource placeholders are editable or replaceable when the deck is instructional or practice-oriented.

Include README instructions for starting, editing text, replacing images, switching modes, exporting PDF/PNG/PPTX, importing/exporting `deck.json`, restoring defaults, changing theme colors, and adjusting aspect ratio.

### 7. Export And QA

Required commands in generated projects:

```txt
npm run dev
npm run build
npm run preview
npm run export:pdf
npm run export:png
npm run export:pptx
npm run validate
npm run screenshot
```

Codex must run the final project before delivery:

- Install dependencies if the generated project does not already have them.
- Run `npm run build`, `npm run validate`, and `npm run screenshot`.
- Verify the active URL serves the newly created workspace, not an older project. If using localhost, prefer a fresh port or clearly confirm the served project root/logs match the new output directory.
- Verify the deck in a clean browser context or with versioned storage behavior so stale localStorage cannot hide missing generated images or resurrect old placeholders.
- Run export commands when the user requested concrete export files, or when export fidelity is part of the acceptance criteria.
- Verify exported PNG dimensions are 16:9 and exported PPTX uses 16:9 pages. If a browser-side download path exists, it must also use fixed 16:9 slide screenshots, not responsive viewport captures.
- Start or verify a local dev/preview server for Vite/React projects and provide the working localhost URL in the final response. If the deck is a static HTML-only project, verify it can be opened directly and provide the absolute HTML path.
- Do not ask the user to run setup, build, validate, screenshot, preview, or export commands as the normal handoff path. Terminal commands may be documented in README for later maintenance, but the delivered result must already be built, QA-checked, and runnable.
- If sandboxing, dependency installation, browser launch, or port conflicts block running the project, try the appropriate approval/escalation path, choose another port when needed, and clearly report any remaining blocker instead of pretending the project was run.

Generate `qa-report.md`. Check tiny text, overflow, missing images, safe margins, visual consistency, one core point per slide, content depth, large-format layout, non-corner visual placement, practice upload/QR closing when applicable, edit/preview/present/export modes, image replacement, browser-side one-click PPTX export, command-line PDF export, PNG export, and command-line PPTX export. Fix issues automatically when possible.

## Resource Guide

- `scripts/create-style-preview.ts`: create `styleframe.html` and `styleframe.css` from a visual direction.
- `scripts/export-style-preview.ts`: screenshot the style board into `style-preview.png`.
- `scripts/create-deck-project.ts`: scaffold the editable HTML deck project after approval.
- `scripts/export-pdf.ts`, `scripts/export-png.ts`, `scripts/export-pptx.ts`: export final HTML slides.
- `scripts/validate-deck.ts`: check deck data, text density, assets, and required commands.
- `scripts/screenshot-check.ts`: render screenshots and flag layout risks.
- `references/slide-patterns.md`: choose slide types.
- `references/component-system.md`: choose editable, visual, diagram, and technical components.
- `references/export-guide.md`: export architecture and priorities.
- `references/image-generation-guide.md`: rules for local visual assets.
- `references/layout-content-rules.md`: avoid thin content, small-island layouts, repeated corner thumbnails, and missing practice upload endings.
- `references/reference-layout-systems.md`: reusable layout systems distilled from polished reference decks, including dark tech pitch, warm green brand deck, architectural investment, bright AI platform, and financial annual report styles.
- `references/socratic-briefing.md`: 10-20 question clarification workflow for ambiguous deck requests before visual generation.
- `references/quality-checklist.md`: final QA checklist.
- `agents/*.md`: role-specific guidance for content, visual, frontend, export, and QA passes.

## Example Invocation Flow

1. User: "Help me make a better 10-slide Codex intro deck in a technology launch style."
2. If key details are missing, ask 10-20 Socratic clarification questions and wait for answers.
3. Create `outline.md` and `visual-direction.md`.
4. Create `styleframe.html`, `styleframe.css`, and `style-preview.png`.
5. Ask the user to approve or revise the style preview before continuing.
6. If approved, create `visual-lock.json`, `content-lock.md`, and `deck-project/`.
7. Install/build as needed, run validation and screenshots, start or verify the project locally, run requested exports, and produce `qa-report.md`.

---
> Source: [ziguishian/better-ppt-html-deck](https://github.com/ziguishian/better-ppt-html-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
