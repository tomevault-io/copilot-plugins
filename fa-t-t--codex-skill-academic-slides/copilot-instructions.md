## codex-skill-academic-slides

> Create academic PPT decks, proofsheet PPTX files, poster-style slides, one-slide system architecture diagrams, and paper-report slides as 16:9 image-native presentations using GPT Image 2 through the local `gpt-image-2-skill` backend, with reference-first handling for real figures from txt/md/docx/tex folders and figure directories. Use when the user asks for PPT, slides, presentation slides, report deck, proofsheet, poster, system architecture diagram, academic presentation, poster-like slide, conference talk visuals, or Nature/Science-style white-background slide images with coherent story flow and reusable files under a `slides/` directory.


# PPT Image

This is the canonical PPT-generation skill. It replaces HTML/Beamer/tcbposter-first slide workflows with an image-native workflow: Codex plans the story, writes a strict visual prompt for every slide, generates each 16:9 slide image through `gpt-image-2-skill`, reviews the raster output, then packages the accepted images into a PPTX.

The backend is the installed `gpt-image-2-skill` CLI, normally at `$CODEX_HOME/skills/gpt-image-2-skill/scripts/gpt_image_2_skill.cjs`. Resolve the current skill directory from the loaded `SKILL.md`; commands below use `$PPT_IMAGE_SKILL_DIR` for this skill and `$GPT_IMAGE_2_SKILL_DIR` for the image backend. Do not use `ppt-design`, Beamer, tcbposter, Gemini, Paperbanana, or python-pptx layout reconstruction as the primary rendering route.

If shell commands are needed, set `CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"`, `PPT_IMAGE_SKILL_DIR="$CODEX_HOME/skills/ppt-image"`, and `GPT_IMAGE_2_SKILL_DIR="$CODEX_HOME/skills/gpt-image-2-skill"` unless the skill is installed under different names.

## Hard Boundary

Image generation is not a truth engine. Never ask GPT Image 2 to invent data, citations, bar heights, equations, or experimental numbers. Extract claims and numbers from source files first, lock them in a content manifest, and make the slide prompt reproduce only that manifest. For quantitative plots, use an existing plot as a reference image or create a plain data figure outside the image model, then ask GPT Image 2 to compose or restyle around it while preserving geometry. For formulas, keep only one short core formula per slide; if exact formula rendering fails after retries, stop and report the failure rather than silently accepting a wrong formula.

The default policy is reference-first. If the input contains real figures, plots, diagrams, screenshots, or paper assets, preserve them unless there is a clear reason to redraw. The image model may create surrounding slide layout, explanatory panels, and conceptual visuals, but exact author-provided figures should be inserted as original pixels whenever the slide's correctness depends on them. Use GPT Image 2 to generate a replacement figure only when no suitable source figure exists or when the user explicitly asks for a reinterpretation.

## Output Contract

For every deck, create a self-contained project-local folder:

```text
slides/<slug>_<YYYYMMDD_HHMMSS>/
  source_text.md
  source_inventory.json
  content_manifest.md
  storyboard.md
  style_bible.md
  deck_manifest.json
  assets/
    reference_images/
    original_assets/
  prompts/
    slide_01.md
    slide_02.md
  images/
    slide_01_base.png
    slide_01.png
    slide_02.png
  composites/
    slide_01_composite.json
  review/
    slide_01_review.md
    deck_review.md
  outputs/
    deck.pptx
    contact_sheet.png
    proofsheet.pptx
```

Keep all generated files in this folder. Do not scatter slide images in `figures/`, `outputs/`, or temporary directories unless the user explicitly asks.

## Inputs

Accept three common input shapes without asking the user to reformat:

```text
1. A plain information document: .txt, .md, or .docx.
2. A single source file: .tex, optionally with neighboring figures.
3. A project folder: TeX/Markdown/Word files plus fig/, figures/, images/, or result assets.
```

For Word documents, extract document text and embedded media. For TeX folders, parse text and `\includegraphics{...}` references, then gather figure assets. For plain text or Markdown with no figures, use the content alone and let the agent generate visual structure from the scientific story.

## Template Selection

Before writing the storyboard, choose exactly one primary template and load only that template file from `$PPT_IMAGE_SKILL_DIR/templates/`.

Use `templates/complete_deck.md` for a full PPTX, report deck, paper presentation, group meeting deck, or conference talk.

Use `templates/proofsheet.md` when the user wants a proofsheet, slide overview, visual QA deck, or compact review PPTX from existing slide images. This mode should package existing images and avoid regenerating content unless the user asks for fixes.

Use `templates/poster.md` for a poster, one-page research summary, graphical abstract poster, or poster-style slide.

Use `templates/architecture_diagram.md` for a single system architecture diagram, method pipeline, software stack, experimental workflow, or one-slide technical schematic.

If the request mixes modes, choose the deliverable the user names as final. For example, a full PPTX may also include a proofsheet as a QA artifact, but the primary template remains `complete_deck.md`. Do not combine template constraints in a way that makes the prompt overdetermined.

## Default Visual Style

Use a clean academic style that resembles high-end Nature or Science review graphics: white background, light tinted panels, restrained contrast, thin dark text, generous whitespace, no decorative gradients, no dark hero sections, no marketing card stacks, no cartoon icons, no fake journal logos. Use one coherent style bible for the entire deck.

Default palette:

```text
background: #FFFFFF
ink: #172033
muted_ink: #5B667A
primary_blue: #2F6F9F
teal: #3A8F8A
soft_green: #DFF2EA
soft_blue: #EAF4FA
soft_red: #F8E8E6
accent_red: #B65B5B
grid_gray: #E8EDF2
```

Use 16:9 slides at `2560x1440`, `quality high`, `format png`. Treat `3840x2160` as experimental and use it only when the user explicitly needs 4K. Put size, format, and quality in CLI flags, not in the prompt.

## Prompting Rules

Follow the GPT Image 2 prompting guide from the OpenAI cookbook. If a local `gpt-image` or `gpt-image-2-skill` reference is installed, load its cookbook reference; otherwise use the public GitHub copy at `https://github.com/FA-T-T/gpt_image_2_skill/blob/main/skills/gpt-image/references/openai-cookbook.md`.

The prompt contract must follow three cookbook anchors:

```text
Section 2, Prompting Fundamentals:
Use a skimmable, labeled prompt ordered as goal -> subject/artifact -> key details -> constraints. Specify composition, typography, preservation rules, and iteration deltas. Put literal slide text in quotes. Use high quality for dense text, diagrams, charts, formulas, and high-resolution deck assets.

Section 4.9, Scientific / Educational Visuals:
Write the prompt like an instructional design brief. Define audience, lesson objective, visual format, required labels, required components, scientific constraints, clean flat visual system, clear arrows, readable labels, and enough white space for quick scanning.

Section 4.10, Slides / Diagrams / Charts / Productivity Images:
Write the prompt like an artifact specification, not an illustration request. Name the exact deliverable, define canvas and hierarchy, provide real text or data, describe the visual language, require readable typography and polished spacing, and reject decorative clutter or generic stock-photo treatment.
```

Every slide prompt must be structured in this order:

```text
Goal:
Create one 16:9 academic presentation slide for [deck purpose]. Intended use: [research group meeting / conference talk / paper report / poster-style summary].

Exact artifact:
Deliverable type: [slide / workflow diagram / chart slide / scientific explainer / poster-style summary].
Canvas: 16:9 landscape slide, generated via CLI size flag.
Audience: [specialists / cross-domain researchers / students / committee].
Objective: [what the viewer should understand after 5 seconds].

Slide role:
[cover / problem / theory / method / evidence / limitation / takeaway]

Hierarchy and composition:
[top-left title, main figure region, evidence strip, footer hook, negative space, chart placement, formula placement]

Visual language:
Clean white background; Nature/Science-style academic layout; flat scientific visual system; light palette; thin rules; sharp vector-like rendering; clear sans-serif typography; polished spacing; no logo, no watermark, no fake citation.

Locked text:
Use only these exact text strings, preserving spelling and numbers:
"..."
"..."

Locked data and labels:
Use only these exact numbers, axes, labels, legends, and footnotes:
"..."
"..."

Required scientific components:
[explicit list of modules, arrows, molecules/variables/operators, panels, axes, or table fields that must appear]

Scientific content constraints:
[facts that must be preserved; numbers; formulas; no invented data]

Typography constraints:
[font style, title size relationship, label readability, formula treatment, exact placement for small text]

Reference images:
If reference images are used, list them as Image 1, Image 2, etc. Describe exactly what each provides and what must be preserved.

Preserve / change constraints:
[for edits or iterative regeneration: preserve layout, palette, text, chart geometry, arrows, labels; change only the named issue]

Continuity hook:
[optional small footer or visual carry-over from previous/next slide]

Negative constraints:
No extra text, no fake labels, no unreadable microtext, no decorative background, no heavy shadows, no cartoon style, no hallucinated equations, no invented citations, no stock-photo treatment.
```

For text-heavy slides, keep visible text under 60 words. Put literal text in quotes. Use `quality high` for small text, dense labels, equations, charts, legends, axes, footnotes, or multi-panel slides. Use `quality low` only for cheap style exploration before the locked-text pass; never use low quality for final academic slides.

For scientific explainer slides, require the prompt to name the audience and lesson objective. For chart or evidence slides, require the prompt to name the exact deliverable, give the real data, and specify chart hierarchy. For method diagrams, require every arrow source, target, and label to be written explicitly. For slides with people or real scenes, specify scale, body framing, gaze, action, lighting, and placement; otherwise avoid people entirely in academic decks.

## Workflow

First prepare the source bundle. If the input is a file or folder, run `scripts/prepare_deck_sources.py` before writing the content manifest:

```bash
python3 "$PPT_IMAGE_SKILL_DIR/scripts/prepare_deck_sources.py" \
  --input /absolute/path/to/source_or_folder \
  --deck-dir /absolute/path/to/slides/<deck>
```

If system `python3` cannot import optional PDF-rendering dependencies, call `load_workspace_dependencies` and rerun with the bundled Python executable. The helper writes `source_text.md`, `source_inventory.json`, copies raster figures to `assets/reference_images/`, copies original vector assets to `assets/original_assets/`, extracts media from `.docx`, and renders first-page PNG previews for PDF figures when PyMuPDF is available.

Then extract the content. Read `source_text.md`, `source_inventory.json`, current project files, paper, report, results, or user notes and write `content_manifest.md`. Separate confirmed facts from interpretation. If the user wants a research report, challenge weak framing and state the honest claim boundary before making slides.

Then write `storyboard.md`. Each slide needs one message, a role, the exact visible text, required visuals, source facts, speaker intent, and an optional continuity hook. Consecutive slides that depend on each other should carry a subtle hook such as a repeated small phrase, a shared mini-axis, or a right-edge preview label. Do not use hooks as decoration; they must help narrative flow. Each slide must also declare its visual source mode: `preserve-source-figure`, `reference-guided-redesign`, or `agent-generated`.

Use `preserve-source-figure` when the original figure contains exact data, plots, formulas, architecture diagrams, or author-specific visual intent. Use `reference-guided-redesign` when the original figure is conceptually useful but can be restyled. Use `agent-generated` only when no useful source figure exists.

Then write `style_bible.md`. Fix palette, typography, panel style, chart treatment, formula treatment, and recurring motifs. The style bible is a contract for all subagents and all prompts.

Then write one prompt file per slide under `prompts/`. Prompts must cite their source facts by filename inside the prompt comments or in `deck_manifest.json`; do not rely on memory. For slides that use real figures, the prompt must include a `Source figure plan` section naming the exact asset path, why it is used, and whether it will be preserved by deterministic compositing or used as a GPT Image 2 reference.

Before generation, audit every prompt against `deck_manifest.json`. A prompt is not ready unless it contains `Goal`, `Exact artifact`, `Audience`, `Objective`, `Hierarchy and composition`, `Visual language`, `Locked text`, `Locked data and labels`, `Required scientific components`, `Scientific content constraints`, `Typography constraints`, and `Negative constraints`. If it uses reference images, it must also contain `Reference images` and `Preserve / change constraints`.

Then generate images. If the user explicitly permits subagents, spawn one worker per independent slide or per small slide batch. Tell each worker that it is not alone in the codebase, that it owns only its assigned `prompts/slide_XX.md`, `images/slide_XX.png`, and `review/slide_XX_review.md`, and that it must not edit shared manifests. If the user has not explicitly permitted subagents in the current request, generate slides sequentially in the main agent.

Canonical generation command:

```bash
cd "$GPT_IMAGE_2_SKILL_DIR"
node scripts/gpt_image_2_skill.cjs --json --json-events \
  --provider auto \
  images generate \
  --prompt "$(cat /absolute/path/to/slides/<deck>/prompts/slide_01.md)" \
  --out /absolute/path/to/slides/<deck>/images/slide_01.png \
  --format png \
  --quality high \
  --size 2560x1440
```

For reference-based slide generation, use `images edit` with `--ref-image` and require the prompt to preserve the referenced plot geometry, labels, axes, and relative positions unless explicitly changing them.

For exact figure preservation, generate a base slide with a blank figure aperture, save it as `images/slide_XX_base.png`, then composite the original figure deterministically with `scripts/composite_refs.mjs`. Do this for plots, tables, result images, screenshots, and dense diagrams where exact pixels matter more than stylistic unification:

```bash
node "$PPT_IMAGE_SKILL_DIR/scripts/composite_refs.mjs" \
  --spec /absolute/path/to/slides/<deck>/composites/slide_XX_composite.json
```

The composite spec uses fractional slide coordinates:

```json
{
  "base": "../images/slide_04_base.png",
  "out": "../images/slide_04.png",
  "canvas": [2560, 1440],
  "placements": [
    {
      "image": "../assets/reference_images/result_plot.png",
      "box": [0.10, 0.22, 0.55, 0.58],
      "fit": "contain",
      "background": "#FFFFFF",
      "border": "#E8EDF2"
    }
  ]
}
```

Use GPT Image 2 `images edit --ref-image` only when preserving general style, shape, or semantic content is enough. Do not trust image editing for exact numeric plots when deterministic compositing is possible.

After generation, review every image visually. Check exact text, numbers, formula spelling, visual hierarchy, white-background style, continuity, and whether the slide can be understood in five seconds. Also check the cookbook-specific obligations: audience/objective visible in the design logic, required scientific components present, arrows clear and correct, chart data not fabricated, labels readable, spacing polished, no decorative clutter, and no generic stock-photo treatment. Save the review in `review/slide_XX_review.md`.

When a slide fails, do not append vague instructions to the original prompt. Create a short refinement prompt that names the accepted image as the reference, states `change only ...`, repeats the locked text/data/preservation list, and fixes at most three issues. Regenerate a slide up to three times with targeted corrections. Do not accept a slide with wrong numbers, wrong equations, fake citations, invented chart data, wrong arrow direction, or illegible text.

Finally package the accepted PNG images into a PPTX with `scripts/pack_image_deck.mjs`:

```bash
node "$PPT_IMAGE_SKILL_DIR/scripts/pack_image_deck.mjs" \
  --images /absolute/path/to/slides/<deck>/images/slide_*.png \
  --out /absolute/path/to/slides/<deck>/outputs/deck.pptx \
  --contact-sheet /absolute/path/to/slides/<deck>/outputs/contact_sheet.png \
  --proofsheet-pptx /absolute/path/to/slides/<deck>/outputs/proofsheet.pptx
```

Call `load_workspace_dependencies` first when the bundled Node path is unknown. If the Node package path is not auto-detected, set `CODEX_NODE_MODULES` to the bundled `node_modules` path returned by that tool. The older `pack_image_deck.py` helper is kept only as a fallback when `python-pptx` is available.

The PPTX is intentionally image-based for fidelity. The source of truth is the slide folder: manifests, prompts, images, reviews, and packaged deck.

## Modes

For `slides`, `PPT`, `complete PPTX`, or `report deck`, use `templates/complete_deck.md` and create a multi-slide 16:9 deck. Package `outputs/deck.pptx`, `outputs/contact_sheet.png`, and `outputs/proofsheet.pptx` by default.

For `proofsheet`, use `templates/proofsheet.md` and package existing slide images into `outputs/proofsheet.pptx`. A proofsheet is a review artifact, not a replacement for the final deck.

For `poster`, use `templates/poster.md`; create a single poster-style 16:9 summary slide by default and package it as a one-slide PPTX under the same `slides/<slug>_<timestamp>/` structure. If the user asks for print A0/A1, use a custom GPT Image 2 size that respects the model constraints, and still save it under `slides/`.

For `system architecture diagram`, `architecture`, `pipeline`, or `workflow figure`, use `templates/architecture_diagram.md`; create a single 16:9 technical schematic and package it as a one-slide PPTX.

For `paper illustration`, create one or more figure slides or figure assets under `slides/<slug>_<timestamp>/assets/` and include prompts and reviews. Do not use Gemini or Paperbanana.

## Validation Checklist

Before final delivery, verify that `images/` contains the expected image count, every image is 16:9, the expected PPTX output for the selected template exists, `outputs/contact_sheet.png` exists when requested, `outputs/proofsheet.pptx` exists for complete-deck or proofsheet mode, and `review/deck_review.md` states any residual risks. In the final response, give the PPTX path, the contact sheet or proofsheet path when present, and the slide folder path.

---
> Source: [FA-T-T/codex-skill-academic-slides](https://github.com/FA-T-T/codex-skill-academic-slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
