## makedocskill

> Generate company-specific business documents: board materials, IR / CFO proposals, internal approval documents, Chinese/Japanese bilingual documents, Word/PDF deliverables, and 1920x1080 HTML/PDF slide decks. Runs a Brain -> Router -> Canvas intelligence pipeline (not theme application): infers the real deliverable, rebuilds structure, challenges weak content, locks numbers and language, then renders and verifies real DOCX/PDF/HTML. Two modes: Quick Mode (drop a source file/text -> polished output) and Full Mode (8-step audited pipeline with company tokens, sparring, review, and PDF QA). Depth control S/M/L/XL. Triggered when the user wants to create, refine, or reformat a company document, board pack, IR/CFO proposal, business explanation, policy document, bilingual JP/ZH document, or a formal Word/PDF/slide deck — especially when a source company has its own logo, colors, language requirements, and compliance tone. For HTML visual one-pagers / diagram docs, an optional HTML output mode is available (see Reference Files). Requires document tooling for DOCX/PDF rendering; yt-dlp only if extracting from YouTube in HTML mode.


# make-doc-skill v4 — Company Document Generator

Generate polished, company-specific business documents from reusable structure plus company brand tokens. The primary deliverables are **formal Word/PDF board materials, IR / CFO proposals, bilingual Chinese/Japanese documents, and 16:9 HTML/PDF proposal decks.**

This is **not** a theme applier. It runs a Brain -> Router -> Canvas intelligence pipeline that infers the real deliverable, rebuilds structure, fixes language and diagrams, applies per-company brand tokens, and then exports and verifies real DOCX/PDF/HTML.

**Use when:** the user wants to create, refine, or reformat a board material, internal approval document, IR / CFO proposal, business explanation, operating policy, bilingual JP/ZH document, or any formal Word/PDF/slide deliverable — especially when the source company has its own logo, colors, language requirements, compliance tone, and output folder conventions.

## Reference Files

Methodology (loaded on demand):
- `docs/PIPELINE.md` — the 8-step audited Full-Mode pipeline (01_company_tokens -> 08_pdf_qa)
- `docs/PROMPT_ENGINEERING.md` — Brain/Router/Canvas role design + prompt meta-principles
- `docs/DESIGN_RULES.md` — DOCX / deck / diagram / token / brand-color hard constraints
- `docs/SIZE_DEPTH_MATRIX.md` — S/M/L/XL depth control

Optional **HTML output mode** (loaded only when the requested deliverable is an HTML visual one-pager or diagram doc, not a formal Word/PDF document):
- `templates.md` — 18 HTML visual template skeletons
- `strategies.md` — 28 content strategies + expert questioning chains
- `components.md` — 16 HTML component templates
- `depth-rules.md` — HTML depth-visualization rules

The HTML machinery is preserved as a capable secondary output. The company-document pipeline below is the primary path.

---

## Core Methodology — Brain → Router → Canvas

The skill is a three-layer pipeline, not a single-shot generator. Even when Brain rewrites heavily, Canvas never dumps "analytical prose" into a template — Router forces the translation into document structure.

| Layer | Role | Responsibility | Input | Output |
|-------|------|----------------|-------|--------|
| **Brain** | Editor + domain analyst | Understand the real deliverable, rebuild structure, challenge weak content, fix language. This is the **Codex Intelligence Layer**. | Source material + company context | Structured, corrected content + section plan |
| **Router** | Document architect | Map deliverable type -> output family (formal DOCX / split bilingual / 16:9 deck / HTML) -> section schema + renderer + template | Brain output | Composition plan + renderer choice |
| **Canvas** | Renderer + typesetter | Apply tokens, layout rules, diagram rules; render DOCX/HTML; export PDF. Makes **no** editorial decisions. | Composition plan + tokens | DOCX/PDF/HTML + QA |

**Core principle: depth comes from the sharpness of judgment, not the number of steps.** A board document is convincing because Brain read the material correctly and rebuilt it — not because Canvas applied a color theme. Tokens are the design memory; Brain judgment is the execution engine.

**Prompt meta-principles** (apply when writing any sub-prompt in this skill):

1. **Role > Instruction** — say "You are the CFO's board-document editor at a TSE-listed company," not "please improve this."
2. **Constraint > Freedom** — explicit prohibitions ("no fabricated figures") beat vague encouragement.
3. **Structure > Prose** — make the model fill a composition table, do not let it free-write the document.
4. **Few-shot > Description** — one correct DOCX/slide example anchors quality better than ten rules.
5. **Segmented > One-shot** — the pipeline exists so each step's prompt is short, focused, and auditable.

## Two Modes — Quick Mode / Full Mode

| Mode | Trigger | Flow |
|------|---------|------|
| **Quick Mode** | User drops a source (existing Word/PDF/text) and wants a fast polished output, no project setup | Detect input -> extract -> infer real deliverable -> apply company tokens (or sensible default) -> Brain light pass -> render -> export PDF -> report path |
| **Full Mode** | Board/IR material, bilingual delivery, or "build the document properly" | 8-step audited pipeline with intermediate files and user checkpoints (see `docs/PIPELINE.md`) |

Quick Mode trades sparring depth for speed, but **still runs numerical-fidelity and language-lock checks and never fabricates**. After a Quick output, offer "upgrade to Full Mode" (keeps the normalized source, adds tokens/sparring/review).

## Codex Intelligence Layer

Design tokens alone are not enough. They define colors, fonts, sizes, and spacing, but they do not decide whether a document is actually convincing, readable, or suitable for a Japanese listed-company context. This layer **is the Brain** in the pipeline above.

Use this layer every time:

- Infer the real deliverable, not only the literal file type. If the user asks for a Word/PDF, identify whether it is board material, IR material, CFO proposal, business explanation, or public-facing deck.
- Read the existing document before designing. Extract not only text, but also weak points: broken hierarchy, bad line breaks, crowded tables, unstable diagrams, wrong alignment, poor page rhythm, and mismatched tone.
- Convert vague brand direction into concrete tokens. For example, "do not use blue; use our corporate colors" means defining the brand's actual system (e.g. red/gray) and removing accidental blue from headings, tables, links, diagrams, and charts.
- Make editorial decisions. Rewrite headings, split overloaded paragraphs, normalize terminology, and tighten section order when the document needs it.
- Use document-native judgment. Formal DOCX needs chapter page breaks, readable 12pt body text, controlled tables, and stable TOC. A 16:9 CFO proposal needs visual pacing, slide hierarchy, and full-screen readability.
- Treat diagrams as communication tools, not decoration. Replace messy flow graphics with clean fixed SVG/PNG diagrams only when the diagram clarifies the business mechanism.
- Verify the real output surface. A generated DOCX is not done until the exported PDF is readable. An HTML deck is not done until browser rendering and PDF export both look correct.
- Preserve user file discipline. Do not overwrite source files. Put final files where the user expects them, especially the handoff folder (default ~/Downloads) or next to the original file when requested.
- Prefer decisive improvement over mechanical conversion. If the source was made by another model and is visibly weak, use the tokens as a base, then actively rebuild layout, hierarchy, language, and diagrams.
- Report only what matters: output paths, format status, unresolved assumptions, and any checks that could not be completed.

In short: tokens are the design memory; Codex-style judgment is the execution engine. The skill should inspect, decide, rebuild, verify, and deliver.

## Sparring, Anchoring & Prohibition Rules

This is what makes the Codex Intelligence Layer reliable rather than just "an LLM rewriting." Run sparring at M depth and above; run anchoring and prohibition rules at all depths including Quick Mode.

### Three-dimension expert questioning (pipeline Step 4)

| Dimension | Goal | Questions to force |
|-----------|------|--------------------|
| **Hypothesis challenge** | Test the core claim | Is the main claim defensible? Is the evidence primary data or assertion? What is the strongest counter-argument a board member or auditor would raise? |
| **Blind-spot completion** | Fill missing dimensions | What is missing — downside, risk, dilution, regulatory, tax, precedent, timeline? What will the audience ask that the draft does not answer? |
| **Perspective reconstruction** | Offer a better framing | Would risk-first / comparison-first / timeline-first framing serve this audience better than the current order? |

Sparring rounds by depth: S = 0–1, M = 1–2, L = 2–3, XL = 3+ (XL adds an extreme test: "what if the core assumption is wrong?").

### Anchoring rules (non-negotiable for listed-company material)

- **Numerical fidelity** — use figures exactly as in the source. Never round ¥2.4B to "about ¥2B," never invent a number. Currency symbols, units, and fiscal periods stay exact.
- **Material anchoring** — only state what the source supports. Mark missing data `[DATA_GAP]` rather than filling it.
- **Source attribution** — mark supplemental/background knowledge as `[Supplemental]` so it is distinguishable from sourced claims.
- **Language lock** — once the output language is set (Step 2), the whole chain obeys it. Quotations keep their original language; `03_source_normalized.md` preserves source language; the DOCX language attribute and fonts follow the output language.
- **Pyramid + MECE** — conclusion first, then supporting arguments, then data. Sections must be mutually exclusive and collectively exhaustive.

### Prohibited output (Brain pipeline)

- No vague filler ("this area is evolving rapidly," "we must monitor future developments") used to avoid stating a conclusion.
- No judgment avoidance — if the material supports a recommendation, state it.
- No fabricated figures, citations, legal/tax conclusions, or approvals. Unverified legal/accounting/tax statements must be explicitly marked draft/assumption.
- No phenomenon-listing without causal explanation.
- No surface-level paraphrasing of a weak source — rebuild it.

## Required Inputs

Collect these before rendering:

- Company identity: company name, ticker, market, logo, corporate colors, official website, IR tone.
- Output type: DOCX/PDF, HTML/PDF deck, bilingual combined document, or split language files.
- Audience: board, CFO, IR, operating team, external partner, regulator, or public investor.
- Source content: Markdown, outline, existing Word/PDF, pasted notes, or bilingual draft.
- Language mode: Japanese, Chinese, bilingual side-by-side, or split Japanese/Chinese outputs.
- Delivery folder: default to ~/Downloads unless the user specifies another path.

If the source file is in Dropbox, Google Drive, Downloads, or a user handoff folder, read it as source material only unless the user explicitly asks to modify it.

## Token Model

Every company should get a token file before rendering. Use JSON or YAML. Do not hardcode colors, fonts, sizes, or spacing directly into templates unless the token system explicitly allows it.

Minimal token shape:

```yaml
company:
  legal_name: "Acme Holdings, Inc."   # example only — replace with the target company
  ticker: "0000"
  market: "Tokyo Stock Exchange Growth"
  brand_name: "Acme"

brand:
  logo_path: "assets/company-logo.png"
  primary: "#c0392b"        # example brand palette — replace with the company's real tokens
  primary_dark: "#922b21"
  primary_light: "#e74c3c"
  primary_bg: "#fdecea"
  text: "#242424"
  text_muted: "#666666"
  text_subtle: "#9a9a9a"
  background: "#ffffff"
  background_pale: "#fafafa"
  section: "#f3f3f3"
  section_alt: "#e9e9e9"
  border: "#d9d9d9"
  border_strong: "#bdbdbd"

language:
  ja_font: "Hiragino Sans"
  zh_font: "PingFang SC"
  latin_font: "Aptos"
  mono_font: "JetBrains Mono"

docx:
  page_size: "A4"
  margin_mm: 18
  body_pt: 12
  line_spacing: 1.18
  chapter_page_break: true
  toc_alignment: "left"
  page_number_alignment: "center"

slides:
  canvas_px: [1920, 1080]
  base_font_px: 20
  print_page_mm: [508, 285.75]
```

For a 1920x1080 HTML slide deck, use the same design-token discipline (example palette below — replace per company):

```css
:root {
  --fs-display: 3.75rem;  /* 75px */
  --fs-title: 1.5rem;     /* 30px */
  --fs-heading: 1.1rem;   /* 22px */
  --fs-body: 0.9rem;      /* 18px */
  --fs-caption: 0.8rem;   /* 16px */

  --primary: #c0392b;
  --primary-dark: #922b21;
  --primary-light: #e74c3c;
  --primary-bg: #fdecea;
  --text: #272727;
  --text-muted: #737373;
  --text-subtle: #b2b2b2;
  --white: #ffffff;
  --bg-pale: #fafafa;
  --bg-section: #f3f3f3;
  --bg-section-alt: #ebebeb;
  --border: #dddddd;
  --border-strong: #c5c5c5;
}
```

Rules:

- Use only tokenized font sizes for slide text.
- Use company primary color as the only functional accent unless the company token set explicitly defines additional semantic colors.
- Avoid blue/green/orange functional colors when the company brand does not call for them.
- Use hierarchy through weight, spacing, and muted text, not uncontrolled font sizes.
- Keep color variables in :root; avoid hardcoded hex values in components.

## Depth Control — S / M / L / XL

Depth controls how hard Brain analyzes and how rigorous QA is. It does **not** control whether numerical fidelity or language lock apply — those always apply. Full matrix in `docs/SIZE_DEPTH_MATRIX.md`.

| Dimension | S | M | L | XL |
|-----------|---|---|---|-----|
| **Use case** | 1-page exec summary / memo | Standard proposal / explanation | Full board material / IR document | Comprehensive board pack + appendix |
| **Section count** | 3–5 | 5–8 | 8–12 | 12+ |
| **Brain depth** | Core conclusion + actions | + evidence evaluation | + explanatory framework + alternatives | + multi-perspective review + extreme test |
| **Sparring rounds** | 0–1 | 1–2 | 2–3 | 3+ |
| **Diagrams** | 0–1 | 1–2 | 2–4 | 4+ |
| **QA rigor** | Output exists + fonts + path | + TOC + pagination | + bilingual parity + diagram QA | full Quality Gates + visual contact sheet |
| **Deck slides (if 16:9)** | 3–6 | 6–10 | 10–15 | 15+ |

If the user does not specify a size, infer it from the deliverable: a memo is S, a CFO proposal is M–L, a full board material with appendix is L–XL.

## Output Families

### Formal DOCX/PDF (primary)

Use for board packs, internal proposals, policy explanations, and Japanese listed-company materials.

Rules:

- Body text must default to 12pt unless the user specifies otherwise.
- Japanese body font: Hiragino Sans or another approved Japanese corporate font.
- Chinese body font: PingFang SC or another approved Chinese corporate font.
- Latin/numeric font: Aptos, Arial, or the company-approved Latin font.
- Each chapter starts on a new page for formal board materials.
- Table of contents entries are left-aligned; page numbers should be visually consistent and centered or right-tabbed according to the chosen style.
- Tables should use restrained borders, company accent header fills, and readable 10.5-11.5pt table text depending on density.
- Do not use blue unless blue is part of the company brand tokens.
- Avoid Word freeform flow arrows for complex diagrams; render diagrams as fixed SVG/PNG images and place them into the DOCX.

### Split Chinese/Japanese Documents

Use when the user asks to separate languages.

Rules:

- Generate one Japanese DOCX/PDF and one Chinese DOCX/PDF.
- Do not leave Chinese fragments in Japanese output or Japanese fragments in Chinese output unless they are proper nouns.
- Keep page structure, chart numbering, tables, and section hierarchy parallel across languages.
- Use language-specific fonts and proofread line breaks independently.

### 1920x1080 HTML/PDF Decks

Use for CFO proposals, IR decks, operating briefings, and large-screen review.

Base assumptions:

- Canvas: 1920x1080 px.
- html { font-size: 20px; }
- 1rem = 20px.
- Target: conference room readability at 1-3m and PDF export.
- Delivery: single HTML file plus PDF, zero build if possible.

Slide layout:

- Header accent bar: 6px using company primary color.
- Title band near top, content area below, footer fixed at bottom.
- Default content padding: 40px 88px.
- Compact content padding: 28px 72px.
- Tight content padding: 20px 64px.
- Footer height: about 48px.
- Use grids from 2 to 6 columns only; avoid 7+ column grids.
- Cards should have minimum useful width around 250px.
- Border radius should stay restrained, typically 6px or less.

Print/PDF:

```css
@page {
  size: 508mm 285.75mm;
  margin: 0;
}
```

Rules:

- Do not use transform: scale() to fix print layout.
- Print with background graphics enabled.
- Verify PDF page count and page size after export.

### HTML Visual Document (optional secondary mode)

For an HTML one-pager / diagram doc / visual explainer (not a formal company document), use the v3 HTML machinery in the reference files: `templates.md` (18 templates), `strategies.md` (28 strategies), `components.md` (16 components), `depth-rules.md`. That path produces a self-contained single-page-scroll or 16:9 HTML document. Apply company tokens instead of the default blue CI when a company is specified.

## Document Schema

Use a content schema so company-specific rendering is repeatable:

```yaml
document:
  id: "sample-board-material"
  title_ja: "新中期経営計画の概要に関する件"
  title_zh: "关于新中期经营计划概要的议案"
  document_type: "board_material"
  audience: "board"
  languages: ["ja", "zh"]
  outputs: ["docx", "pdf"]

sections:
  - id: "executive-summary"
    title_ja: "提案の趣旨"
    title_zh: "提案主旨"
    page_break_before: true
    blocks:
      - type: "paragraph"
        body_ja: "..."
        body_zh: "..."
      - type: "table"
        columns_ja: ["項目", "内容"]
        columns_zh: ["项目", "内容"]
        rows: []

diagrams:
  - id: "fund-flow"
    type: "flow"
    render_as: "svg_image_for_docx"
```

### Per-section composition (Step 5 blueprint)

Before rendering, define each section with these fields (the Word/Deck analog of a visual composition table). This forces Router to translate Brain's analysis into document structure rather than prose:

1. **Title** — message-type, conveys the conclusion (not "Background" but "OA settlement is mainstream, but risk skews to the seller").
2. **Core message** — the one thing this section must land.
3. **Block / component type** — heading, paragraph, table, callout, KPI row, diagram.
4. **Token / color allocation** — which brand tokens; accent used sparingly.
5. **Source reference** — which normalized source material backs it.
6. **Logical relation** — how it connects to the previous/next section.
7. **Layout hint** — full-width table, two-column, diagram-left, etc.
8. **Information density** — low / medium / high (do not stack high-density sections consecutively).

## Pipeline — Full Mode 8 Steps with Intermediate Products

Each step writes a reviewable file into the project folder and ends with a user checkpoint: **proceed / redo this step / skip with defaults / jump to render**. Skipping applies sensible company-token defaults. Iteration archives prior files to `_history/v[N]/`; the project root always holds the latest version. Full trilingual detail in `docs/PIPELINE.md`.

```
Step 1  Company tokens     -> 01_company_tokens.md     Brand/lang/docx/slide tokens (Router input)
Step 2  Document brief      -> 02_document_brief.md     Deliverable type, audience, language mode, depth (S/M/L/XL), outputs, delivery folder
Step 3  Source normalized   -> 03_source_normalized.md  Existing Word/PDF/text -> structured MD/YAML; original quotes preserved; [DATA_GAP] marked
Step 4  Codex sparring      -> 04_codex_sparring.md     3-dimension expert questioning, weak-point list, editorial decisions
Step 5  Composition plan    -> 05_composition_plan.md   Section-by-section blueprint (8 fields), renderer + template choice
Step 6  Layout review       -> 06_layout_review.md      Pre-render structure / quality / brand gate
Step 7  Render log          -> 07_render_log.md         DOCX/HTML generated, decisions logged, PDF exported
Step 8  PDF QA              -> 08_pdf_qa.md             pdfinfo / pdffonts + visual checks, gate results, final paths
```

The per-step actions reuse the Workflow below. Mapping: Workflow "Inspect / build tokens" = Steps 1–2, "Normalize content" = Step 3, "Choose renderer" = Step 5, "Render first output" = Step 7, "Run QA" = Step 8, "Place deliverables / report" = post-Step 8.

**Pipeline transparency**: every decision is recorded in its step file (traceable); the user can pause/review/modify at any step (auditable); Brain explains key choices in-line (explainable); iteration is cheap (low-cost rollback).

## Workflow

1. **Inspect source materials.** Identify the current document, language, company, output expectations, and source folder. If an existing polished file exists, treat it as style evidence.
2. **Build or update company tokens.** Extract logo color, accent color, text colors, fonts, spacing, table rules, chart rules, and output placement rules. Keep company tokens separate from document content.
3. **Normalize content.** Convert existing Word/PDF/pasted text into a structured Markdown/YAML source. Preserve headings, tables, references, and legal/compliance wording.
4. **Choose renderer.** Use DOCX rendering for formal documents. Use HTML/CSS rendering for 16:9 decks. Use fixed SVG/PNG images for diagrams inside Word.
5. **Render first output.** Generate DOCX/HTML first, then export PDF from that source. Do not manually edit the PDF.
6. **Run QA.** Inspect page count, fonts, layout, TOC, line breaks, table widths, diagram placement, language residue, and whether brand colors are respected.
7. **Place deliverables.** Put final DOCX/PDF/HTML in the user-requested handoff folder. Default: ~/Downloads.
8. **Report concise result.** Give exact output paths and any remaining risks, such as unverified legal citations or missing official logo files.

## Diagram Rules

Use diagrams only when they teach more than a paragraph or table.

Allowed diagram types: process flow, funds flow, architecture map, timeline, matrix, org tree, layer stack, waterfall, simple KPI chart.

Rules:

- One major diagram per slide or page section.
- For DOCX, create diagrams as SVG/PNG and insert as fixed images.
- For HTML decks, use inline SVG. Keep it self-contained.
- Avoid Mermaid unless the output pipeline reliably converts it into a fixed image.
- Use the primary color for at most 1-2 focal elements.
- Do not make every node red.
- Use neutral fills and borders for normal nodes.
- Keep node count under 9 where possible; split complex diagrams.
- Arrow endpoints must land exactly on node edges.
- Use consistent spacing multiples, preferably 4px/8px.
- Avoid decorative icons, glow, gradients, floating arrows, and non-brand colors.
- For funds flow, prefer a clean left-to-right or top-to-bottom flow with actor lanes, numbered steps, and a short note table.

## Layout Rules For Formal Word Documents

Use these defaults unless the user asks otherwise:

- A4 portrait.
- Body: 12pt. Body line spacing: 1.15-1.2.
- Major heading: 17-20pt. Section heading: 14-16pt. Subsection heading: 12.5-13.5pt.
- Table text: 10.5-11.5pt. Caption/note: 9.5-10pt.
- Chapter starts on new page.
- Use clear hierarchy but avoid excessive decoration.
- Use company primary color for headings, rules, table headers, and small accent bars.
- Use neutral gray for borders and secondary text.
- Keep tables within page width; reduce text, rotate nothing unless unavoidable.
- Check that diagram-heavy pages do not have broken wrapping or overlapping objects.

## Layout Rules For 16:9 Proposal Decks

Use these defaults:

- Size: 1920x1080. Each slide is a fixed viewport section.
- Titles are concise; do not overload title bands.
- Body blocks should fill the slide vertically; do not center sparse content as a substitute for adding useful structure.
- Use flex: 1, stretched tables, split columns, callouts, and notes to fill visual space properly.
- Avoid pure decorative backgrounds.
- Keep the company/product/topic visible in the first viewport.
- Make PDF print match browser rendering.

## Language Rules

Japanese:
- Use board-level Japanese for listed-company documents.
- Prefer concise nouns in headings.
- Use Japanese punctuation and legal/compliance wording consistently.
- Avoid casual marketing language in board materials.

Chinese:
- Use professional Chinese for board/finance context.
- Keep company names, ticker codes, legal terms, and proper nouns stable.
- Avoid literal machine translation when Japanese governance terms require localized explanation.

Bilingual:
- If generating a combined bilingual document, align sections one by one.
- If generating split documents, render independently and QA each language.
- Language lock applies (see Anchoring rules): the chosen output language governs the whole render chain; quotations keep their original language.

## Quality Gates

These are the concrete checks behind Step 6 (layout review) and Step 8 (PDF QA). Run them before final delivery:

- Output exists in the requested folder.
- DOCX opens without repair warnings.
- PDF exports from final DOCX/HTML source.
- Fonts are readable and language-appropriate.
- Body text is 12pt for formal Word documents unless overridden.
- TOC alignment is correct.
- Chapters start on new pages when required.
- Tables do not overflow page margins.
- Diagrams are not broken, cropped, or made from unstable Word shape arrows.
- No unintended blue/green/orange appears when the company brand system is red/gray.
- Chinese/Japanese split files contain the correct language.
- Page count is reasonable and matches TOC after updating fields.
- For HTML decks, browser screenshot and PDF export both show non-overflowing slides.
- For PDF decks, page size is 16:9 and all pages are present.
- Numerical fidelity holds: every figure, unit, currency, and fiscal period matches the source; gaps are marked `[DATA_GAP]`; unverified legal/tax statements are marked draft.

Useful local verification commands:

```bash
pdfinfo "/path/to/output.pdf"
pdffonts "/path/to/output.pdf"
python -m compileall scripts
```

For visual inspection, render PDF pages to images or contact sheets and inspect title pages, diagram pages, dense tables, and final appendix pages.

## Evals

Acceptance cases the skill should pass are in `evals/evals.json`: branded Word refine, CN/JP split, CFO 16:9 deck, fund-flow diagram, numerical-fidelity/language-lock. Run them against real generated outputs, not just prompts.

## Handoff Convention

Unless the user specifies another destination:

- Final files go to the user's Downloads folder — `~/Downloads` (macOS/Linux) or `%USERPROFILE%\Downloads` (Windows) — unless the user specifies another destination.
- If refining a file from Downloads, also place the polished file near the original when requested.
- Keep output filenames close to the source filename plus a clear suffix such as _精修版, _日本語版, _中文版, or _CFO Proposal.
- Do not overwrite source files unless explicitly requested.

## Questions To Ask Only If Needed

Ask the user when the answer cannot be safely inferred:

- Which company logo/color should be authoritative if multiple logos or colors exist?
- Should the document be public-facing IR style or internal board style?
- Should bilingual content be combined or split?
- Is the PDF expected to be A4 formal material or 16:9 deck material?
- Are legal, accounting, or tax statements already approved, or should they be marked as draft assumptions?

Do not pause for minor style choices when a reasonable company-token default is available.

## Future Folder Structure

When converting this into a fully installable folder skill, use:

```
make-doc-skill/  (or company-document-generator/)
  SKILL.md
  references/
    brand-token-discipline.md
    prompt-modules.md         # sparring, anchoring, prohibition prompt blocks
  templates/
    docx-formal-board.yml
    html-1920x1080-proposal.html
    tokens.example.yml
  scripts/
    render_docx.py
    render_html_deck.py
    export_pdf.py
    qa_pdf.py
  evals/
    evals.json
  examples/
    company-board-docx/
    company-cfo-deck/
  html-mode/                  # the v3 HTML machinery, optional
    templates.md  strategies.md  components.md  depth-rules.md
```

Keep SKILL.md focused on workflow and rules. Put long templates, examples, and rendering code into separate files only when the skill is installed as a folder.

---
> Source: [jasonhnd/makedocskill](https://github.com/jasonhnd/makedocskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
