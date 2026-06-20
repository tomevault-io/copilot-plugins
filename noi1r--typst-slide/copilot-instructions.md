## typst-slide

> >


# Typst Slide Workflow (Touying)

Universal skill for academic Typst presentations via Touying. Full lifecycle:
create → compile → review → polish → verify.

**Why Typst over Beamer?** Typst compiles in milliseconds (no multi-pass), has sane error messages, built-in scripting, and automatic package management. Touying is the most mature Typst slide framework with rich theme support.

---

## 0. REFERENCE PREAMBLE

Default template for new presentations. Use this unless the user provides a custom template.

```typst
#import "@preview/touying:0.7.0": *
#import themes.university: *
#import "@preview/cetz:0.4.2"
#import "@preview/fletcher:0.5.8" as fletcher: node, edge
#import "@preview/theorion:0.5.0": *
#import cosmos.clouds: *
#show: show-theorion

// CeTZ and Fletcher bindings for Touying animations
// Inside CeTZ canvas, use (pause,) tuple syntax — NOT #pause
#let cetz-canvas = touying-reducer.with(reduce: cetz.canvas, cover: cetz.draw.hide.with(bounds: true))
// Inside Fletcher diagram, use bare pause (no #)
#let fletcher-diagram = touying-reducer.with(reduce: fletcher.diagram, cover: fletcher.hide)

// Semantic colors — use throughout for visual consistency
#let positive = rgb("#0173B2")    // blue (correct, advantage)
#let negative = rgb("#DE8F05")    // orange (limitation, drawback)
#let emphasis = rgb("#029E73")    // green (highlight, key finding)
#let neutral = luma(140)          // muted context
#let accent = rgb("#CC78BC")      // additional accent (purple)

// Semantic text helpers
#let pos(body) = text(fill: positive, body)
#let con(body) = text(fill: negative, body)
#let HL(body) = text(fill: emphasis, body)

#show: university-theme.with(
  aspect-ratio: "16-9",
  // handout: true — uncomment for PDF export (collapses #pause subslides)
  config-common(
    handout: true,
    frozen-counters: (theorem-counter,),
  ),
  config-info(
    title: [Title],
    subtitle: [Subtitle],
    author: [Presenter: \[name\]],
    date: datetime.today(),
    institution: [Shanghai Jiao Tong University],
    logo: emoji.school,
  ),
)

#title-slide()
```

**Rules:**
- **Always `"16-9"` aspect ratio** — modern projectors are 16:9.
- **Always `handout: true` for PDF export** — without this, `#pause` creates duplicate subslide pages in the PDF (e.g., a slide with 3 pauses becomes 3 pages). Set `handout: false` only when presenting live with pdfpc/pympress. This is the #1 source of inflated page counts and "empty-looking" slides.
- **NO heading numbering** — never use `numbly` or `#set heading(numbering: ...)`. Numbered headings (1.1, 2.3) make slides look like a textbook. Slide titles should be plain text.
- **Manual outline, not auto-generated** — never use `#outline()` or `#components.adaptive-columns(outline(...))`. These produce book-style TOC with page numbers and dotted leaders. Instead, write a simple bullet list:
  ```typst
  == Outline
  + Topic A
  + Topic B
  + Topic C
  ```
- **Default presenter**: `author: [Presenter: [name]]` and `institution: [Shanghai Jiao Tong University]`. Override only if user specifies otherwise.
- If user provides a custom template or theme: use theirs.
- **Domain macros go inside math, not as top-level `#let`** — avoid `#let GG = $bb(G)$` then `$#GG_1$` (breaks in math context). Instead write `$bb(G)_1$` directly, or define math-safe macros: `#let GG(sub) = $bb(G)_#sub$`.
- **Available built-in themes** (choose based on context):
  - `themes.university` — academic blue, good for seminars and defenses (default)
  - `themes.metropolis` — clean minimalist, popular at tech conferences
  - `themes.dewdrop` — light and fresh, good for workshops
  - `themes.simple` — bare-bones, maximum content control
  - `themes.aqua` — water-color tones, suitable for soft topics
  - `themes.stargazer` — dark elegant, good for keynotes
- **Theorem environments** via `theorion` (v0.5.0): provides `theorem`, `lemma`, `corollary`, `definition`, `example`, `proof` with auto-numbering. Use `config-common(frozen-counters: (theorem-counter,))` to prevent counter increments during animation subslides.
- **CeTZ** (`@preview/cetz:0.4.2`) for diagrams — Typst's TikZ equivalent.
- **Fletcher** (`@preview/fletcher:0.5.8`) for flowcharts and arrow diagrams — built on CeTZ, much easier for node-edge diagrams.
- **Heading-based slide splitting**: `=` creates section pages, `==` creates slides (for most themes at slide-level=2). Use `---` or `#pagebreak()` for manual page breaks.
- **Special heading tags**: `<touying:hidden>` (don't render), `<touying:skip>` (no section page), `<touying:unnumbered>` (no page number), `<touying:unoutlined>` (exclude from outline).
- **NEVER use `#set page(...)`** — Touying resets it. Use `config-page(...)` in the theme `.with()` call instead.

---

## 1. HARD RULES (Non-Negotiable)

1. **Use `#pause` sparingly; always compile with `handout: true`** — Touying's `#pause` creates animation subslides. In live presentation tools (pdfpc), these work as progressive reveals. But in **normal PDF export, each subslide becomes a separate page** — a slide with 3 `#pause` produces 3 PDF pages, with the first showing only partial content. This is the #1 cause of "empty-looking" slides and inflated page counts. **Always use `config-common(handout: true)` by default** to collapse subslides. Only set `handout: false` when the user explicitly wants animation pages. For complex multi-step constructions, prefer separate slides over `#pause`. **Inside CeTZ canvases, use `(pause,)` tuple syntax**. Inside Fletcher diagrams, use bare `pause`. For list-item-by-item reveals, use `#item-by-item[...]`.
2. **Max 2 colored/styled blocks per slide** — more dilutes emphasis. Demote transitional remarks to plain italic.
3. **Motivation before formalism** — every concept starts with "Why?" before "What?".
4. **Worked example within 2 slides** of every definition.
5. **Typst source is the single source of truth** — CeTZ diagrams, content, notation all originate here.
6. **Verify after every task** — compile, check warnings, review PDF.
7. **Telegraphic style** — keyword phrases, not full sentences. Slides are speaker prompts, not manuscripts. Exception: framing sentences that set up a definition or transition. Each bullet item should be ≤2 lines (~15 words) — if longer, split into sub-items or rewrite more concisely.
8. **Every slide earns its place** — each slide must contain at least one substantive element (formula, diagram, table, theorem, or algorithm). A slide with only 3 short bullets and nothing else must be merged or enriched.
9. **Content density guard** — Typst's layout engine is good but not infallible. For slides mixing text and diagrams, always compile and visually verify. Common overflow happens with large CeTZ canvases below long equation blocks.
10. **Reference slide** — the second-to-last slide (before Thank You) must be a **References** slide listing key cited works. Use `#bibliography("refs.bib")` or manual list with `#text(size: 0.8em)[...]`.
11. **Color and contrast standards** — text-background contrast ratio ≥ 4.5:1 (WCAG AA). Never use red+green for binary contrasts. Prefer blue+orange. Semantic color helpers defined in preamble: `pos()` = positive (blue), `con()` = negative (orange), `HL()` = emphasis (green). These are color-blind safe. Limit total palette to 3-5 colors.
12. **Backup slides** — after the Thank You slide, include 3-5 backup slides for anticipated questions. Use `#show: appendix` before the backup section to freeze slide numbering. Content to include:
    - Full proof of the main theorem (if only a sketch was shown)
    - Parameter choices / security analysis details
    - Extended comparison table with additional baselines
    - Experimental setup details
    - Definitions of prerequisites that were assumed known

---

## 2. ACTIONS

Parse `<user request>` to determine which action to run. If no action specified, ask.

### 2.1 `compile [file]`

Typst compiles in a single pass — no multi-pass needed.

```bash
typst compile FILE.typ
```

For live preview during development:
```bash
typst watch FILE.typ
```

Post-compile checks:
- Check exit code (0 = success)
- Read any error/warning output from stderr
- Open PDF for visual verification
- Report: success/failure, any warnings, page count

**If `typst` is not installed**, guide the user:
```bash
# macOS
brew install typst
# or via cargo
cargo install typst-cli
```

### 2.2 `create [topic]`

**Collaborative, iterative presentation creation. Strict phase gates — never skip ahead.**

#### Phase 0: Material Analysis (if papers/materials provided)

**Read first, ask later.** Must understand the content before asking meaningful questions.

- Read the full paper/materials thoroughly
- Extract: core contribution, key techniques, main theorems, comparison with prior work
- Map notation conventions
- Identify the paper's logical structure and which parts are slide-worthy
- Internally note: prerequisite knowledge, natural section boundaries, what could be skipped or expanded

**Do NOT present results or ask questions yet — proceed directly to Phase 1.**

#### Phase 1: Needs Interview (MANDATORY — informed by Phase 0)

Conduct a **content-driven interview** via AskUserQuestion. The questions below are the **minimum required set** — you MUST also add paper-specific questions derived from Phase 0.

**Minimum required questions** (always ask):
1. **Duration**: How long is the presentation?
2. **Audience level**: Who are the listeners?

**Optional question** (ask when the talk is a journal club, defense, or user seems to want rehearsal support):
3. **Speaker notes**: Would you like speaker notes? If yes, `#speaker-note[...]` blocks will be added per slide with telegraphic talking points. Requires `config-common(show-notes-on-second-screen: right)`.

**Content-driven questions** (derive from Phase 0, ask as many as needed):
- **Prerequisite knowledge**: List concrete technical dependencies. Ask which ones the audience knows.
- **Content scope**: Offer the paper's actual components as options. Ask which to emphasize, skip, or briefly mention.
- **Depth vs. breadth**: If the paper has both intuitive overview and detailed constructions, ask which the user prefers.
- **Paper-specific decisions**: E.g., if the paper compares two constructions, ask whether to present both equally or focus on one.

**Guidelines:**
- 3-6 questions total; don't over-ask, don't under-ask.
- If something is obvious from context (e.g., user said "讨论班"), infer rather than ask.

**Slide count heuristic**: ~1 slide per 1.5-2 minutes.

**Timing allocation table:**

| Duration | Total slides | Intro/Motivation | Methods/Background | Core content | Summary/Conclusion |
|----------|-------------|------------------|-------------------|-------------|-------------------|
| 5min (lightning) | 5-7 | 1-2 | 0-1 | 2-3 | 1 |
| 10min (short) | 8-12 | 2 | 1-2 | 4-5 | 1 |
| 15min (conference) | 10-15 | 2-3 | 2-3 | 5-7 | 1-2 |
| 20min (seminar) | 13-18 | 3 | 2-3 | 6-9 | 2 |
| 45min (keynote) | 22-30 | 4-5 | 5-7 | 10-14 | 2-3 |
| 90min (lecture) | 45-60 | 5-6 | 8-12 | 25-35 | 3-4 |

#### Phase 2: Structure Plan (GATE — user must approve before drafting)

Produce a **detailed outline**. For each section:
- Section title
- Number of slides allocated
- Key content points per slide (1-2 lines each)
- CeTZ/Fletcher diagrams planned (brief description)
- Notation to introduce

**Present the plan to the user.** Ask: structure OK? Expand/shrink/cut anything?

**Do NOT proceed to drafting until user approves.**

#### Phase 3: Draft (iterative, batched)

##### 3a. Typst Slide Syntax

**Heading-based slides** (simple, recommended for most content):
```typst
= Section Title

== Slide Title

Content goes here. Use `#pause` for sequential reveals.

- First point
#pause
- Second point (appears on next click)
```

**Explicit `#slide` syntax** (for advanced layouts like multi-column):
```typst
#slide(composer: (1fr, 1fr))[
  Left column content.
][
  Right column content.
]
```

**Key syntax differences from Beamer:**
- `$x^2$` for inline math (same as Beamer's `$x^2$`)
- `$ x^2 + y^2 = z^2 $` for display math (spaces around content trigger display mode)
- `*bold*` instead of `\textbf{}`
- `_italic_` instead of `\textit{}`
- `#text(fill: red)[colored text]` instead of `\textcolor{red}{}`
- Lists use `-` (unordered) or `+` (ordered), with indentation for nesting
- `#table(columns: 3, ..cells)` instead of `\begin{tabular}`
- `#image("path.png", width: 80%)` instead of `\includegraphics`

##### 3b. Writing Style

- **Telegraphic keywords**, not full sentences. Exception: one framing sentence per slide to set context.
- **Formulas and analysis interleave tightly** — define a quantity, then immediately state its cost/property/implication on the same slide.
- **No conversational hedging** — never write "wait, not exactly" or "actually, let me clarify".
- **Use `*bold*` for key terms** on first introduction; use `#pos[...]` for positive properties, `#con[...]` for drawbacks, `#HL[...]` for key findings.

##### 3b-2. Opening and Closing Strategies

**Opening slide (pick one strategy):**
- **Surprising statistic** — a counter-intuitive number that challenges assumptions
- **Provocative question** — something the audience cannot immediately answer
- **Real-world failure/problem** — "System X failed because..." → "How do we prevent this?"
- **Visual demonstration** — show the phenomenon before explaining it

**Closing strategies:**
- **Call-back to opening** — revisit the opening question/problem, now answered
- **3 key takeaways** — numbered, telegraphic, one slide
- **Open question / future direction** — leave the audience thinking
- **Never end on a bare "Thank You"** — the second-to-last content slide should deliver the lasting impression

##### 3c. Mathematical Slide Patterns

**Definition slide:**
```typst
== Definition: Polynomial Commitment

_Why_: We need a way to commit to a polynomial without revealing it.

#definition(title: "Polynomial Commitment")[
  A polynomial commitment scheme is a tuple $(sans("Setup"), sans("Commit"), sans("Open"), sans("Verify"))$ ...
]

Key properties:
- *Binding*: cannot open to a different polynomial
- *Hiding*: commitment reveals nothing about $f$
```

**Theorem/Proof slide:**
```typst
== Main Result

_Informal_: Our scheme achieves linear prover time with logarithmic verification.

#theorem(title: "Main Theorem")[
  For any polynomial $f$ of degree $n$, the prover runs in $O(n)$ field operations
  and the verifier in $O(log n)$.
]

// Proof on the NEXT slide — never cram theorem + proof on one slide
```

**Comparison slide:**
```typst
== Comparison with Prior Work

#table(
  columns: (1fr, auto, auto, auto),
  align: (left, center, center, center),
  table.header[*Scheme*][*Prover*][*Verifier*][*Proof size*],
  table.hline(),
  [Prior work], [$O(n log n)$], [$O(n)$], [$O(n)$],
  [*Ours*], [#pos[$O(n)$]], [#pos[$O(log n)$]], [#pos[$O(log n)$]],
)
```

##### 3d. Content Density Constraints

**Upper bounds (per slide):**
- ≤ 7 bullet points or items
- ≤ 2 displayed equations (more → split)
- ≤ 5 new symbols introduced
- ≤ 2 styled blocks (theorem/definition/alert boxes)

**Lower bounds (per slide):**
- Each slide MUST contain at least one substantive element: a formula, diagram, table, theorem, or algorithm
- A slide with only ≤ 3 short text-only bullets is **too sparse** — merge or enrich
- Pure text-only slides should be ≤ 30% of the total deck

##### 3e. Batch Workflow

- Work in batches of 5-10 slides, following the approved structure
- After each batch, **compile** (`typst compile`) to catch errors early
- After each batch, self-check: notation consistency, density constraints, motivation-before-formalism
- Continue to next batch only after current batch compiles cleanly

##### 3f. Table Best Practices

Typst tables are native and powerful — no external packages needed.

```typst
#table(
  columns: (2fr, 1fr, 1fr, 1fr),
  align: (left, center, center, center),
  stroke: none,
  table.header[*Method*][*Time*][*Space*][*Accuracy*],
  table.hline(),
  [Baseline], [$O(n^2)$], [$O(n)$], [72%],
  [*Ours*], [#pos[$O(n)$]], [$O(n)$], [#pos[*89%*]],
  table.hline(),
)
```

- Always use `stroke: none` + `table.hline()` for academic-style tables (like `booktabs`)
- Column alignment: numbers centered, text left-aligned
- Max 6-7 columns, 8-10 rows per slide
- Highlight best results with `*bold*` or `#pos[...]`
- For wide tables: use `#utils.fit-to-width(1fr)[#table(...)]` to auto-scale

##### 3g. Algorithm Display

```typst
#import "@preview/algorithmic:0.1.0"
#import algorithmic: algorithm

#algorithm({
  import algorithmic: *
  Function("Verify", args: ($pi$, $x$), {
    Cmt[Parse the proof]
    Assign[$c$][$pi.c$]
    If(cond: $e(c, g) = e(pi.h, tau)$, {
      Return[$sans("accept")$]
    })
    Return[$sans("reject")$]
  })
})
```

- Pseudocode ≤ 10 lines per slide
- Highlight critical lines with color
- Input/output clearly stated

#### Phase 4: Figures

- **CeTZ** for geometric diagrams, coordinate plots, annotated figures
- **Fletcher** for flowcharts, state diagrams, node-edge graphs
- Apply CeTZ quality standards (see Section 2.6)

**Data visualization guidelines (same as Beamer skill — projector-optimized):**
- Simplify ruthlessly — remove minor gridlines, label directly on plot
- Enlarge everything — labels, line widths, markers must be readable at distance
- One message per figure — split multi-panel figures across slides
- Color-blind safe palette: blue (#0173B2) + orange (#DE8F05), with line-style differences as redundant encoding
- For data-driven plots: use CeTZ `cetz.plot` or consider generating with Python and including as images

#### Phase 5: Quality Loop (MANDATORY — iterative)

After completing the full draft, enter the quality loop:

```
┌─→ 5a. Compile (typst compile)
│   5b. Self-Review (structure + content + visual)
│   5c. Score (apply rubric)
│   5d. Fix all issues found
└── If score < 90 and round < 3: loop back to 5a
    If score ≥ 90 or round = 3: report to user
```

**5a. Compilation**
- `typst compile FILE.typ` — single pass, check exit code and stderr
- Open PDF for visual inspection

**5b. Self-Review** — re-read the .typ and verify:

*Structure:*
- [ ] Slide count matches plan (±2 tolerance)
- [ ] Logical flow: motivation → background → technique → results → summary
- [ ] No section has >4 consecutive formal slides without a worked example or visual break
- [ ] Transition sentences between major sections

*Content density:*
- [ ] No slide has only ≤3 short bullets with no math/diagram
- [ ] Pure text-only slides ≤ 30% of deck
- [ ] No slide exceeds upper bounds (7 bullets, 2 equations, 5 symbols, 2 blocks)

*CeTZ and visuals:*
- [ ] No label overlaps in diagrams
- [ ] No content overflowing slide boundary
- [ ] Diagrams fit within remaining slide space
- [ ] All computed points lie on their curves (no hardcoded coordinates)
- [ ] Tables fit within slide width
- [ ] Font sizes in diagrams ≥ `0.8em`
- [ ] Consistent styling across all diagrams

*Notation:*
- [ ] Same symbol used consistently throughout
- [ ] Every symbol defined before use

**5c. Quality Score**

Start at 100. Deduct:

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Compilation failure | -100 |
| Critical | Content overflow (equation or diagram) | -20 |
| Critical | CeTZ diagram overflows slide boundary | -15 per diagram |
| Critical | Undefined variable / import error | -15 |
| Major | Content visually clipped in PDF | -10 per slide |
| Major | CeTZ computed points not on curve | -8 per diagram |
| Major | Sparse slide (≤3 items, no math/diagram) | -5 per slide |
| Major | CeTZ label overlap | -5 |
| Major | Missing references slide | -5 |
| Major | Notation inconsistency | -3 |
| Minor | Excessive spacing hacks | -1 |
| Minor | Font size reduction overuse | -1 per slide |

**Thresholds:**
- **≥ 90**: Ready to deliver. Report to user.
- **80-89**: Acceptable. Fix remaining majors if possible.
- **< 80**: Must fix. Loop back and resolve critical/major issues.

#### Post-Creation Checklist (final gate)
```
[ ] Compiles without errors
[ ] All imports resolve (packages downloaded)
[ ] Score ≥ 90
[ ] Every definition has motivation + worked example
[ ] Max 2 styled blocks per slide
[ ] No sparse slides
[ ] CeTZ diagrams visually verified — no overlaps, no overflow
[ ] Tables fit within slide boundaries
[ ] References slide present (second-to-last, before Thank You)
```

### 2.3 `review [file]` or `proofread [file]`

Read-only report, no file edits.

**5 check categories:**

| Category | What to check |
|----------|---------------|
| Grammar | Subject-verb, articles, prepositions, tense consistency |
| Typos | Misspellings, duplicated words, **unreplaced placeholders** (`[name]`, `[TODO]`, template remnants) |
| Overflow | Long equations, too many items per slide |
| Consistency | Citation format, notation, terminology, block usage |
| Academic quality | Informal abbreviations, missing words, claims without citations |

**Report format per issue:**
```
### Issue N: [Brief description]
- **Location:** [slide title or line number]
- **Current:** "[exact text]"
- **Proposed:** "[fix]"
- **Category / Severity:** [Category] / [High|Medium|Low]
```

### 2.4 `audit [file]`

Visual layout audit. Read-only report.

**Check dimensions:**
- **Overflow:** Content exceeding boundaries, wide tables/equations
- **Font consistency:** Inline size overrides, inconsistent sizes
- **Block fatigue:** 2+ styled blocks per slide
- **Spacing:** Manual spacing overuse, structural issues
- **Layout:** Missing transitions, missing framing sentences

### 2.5 `pedagogy [file]`

Holistic pedagogical review. Read-only report.

**13 patterns to validate:**

| # | Pattern | Red flag |
|---|---------|----------|
| 1 | Motivation before formalism | Definition without context |
| 2 | Incremental notation | 5+ new symbols on one slide |
| 3 | Worked example after definition | 2 consecutive definitions, no example |
| 4 | Progressive complexity | Advanced concept before prerequisite |
| 5 | Fragment reveals | Dense theorem revealed all at once |
| 6 | Standout slides at pivots | Abrupt topic jump, no transition |
| 7 | Two-slide strategy for dense theorems | Complex theorem crammed in 1 slide |
| 8 | Semantic color usage | Binary contrasts in same color |
| 9 | Block hierarchy | Wrong block type for content |
| 10 | Block fatigue | 3+ blocks on one slide |
| 11 | Socratic embedding | Zero questions in entire deck |
| 12 | Visual-first for complex concepts | Notation before visualization |
| 13 | Side-by-side for comparisons | Sequential slides for related definitions |

### 2.6 `cetz [file]`

CeTZ diagram review — the Typst equivalent of the Beamer skill's TikZ review.

**Quality standards:**
- Labels NEVER overlap with curves, lines, dots, or other labels
- Visual semantics: solid=observed, dashed=counterfactual, filled=observed, hollow=counterfactual
- Line weights: use `stroke: 1.5pt` for data, `stroke: 1pt` for axes, `stroke: 0.5pt` for grid
- Minimum spacing between labels and graphical elements
- **Mathematical accuracy**: NEVER hardcode y-coordinates for points on curves. Compute from the same function.

**Common CeTZ patterns:**

*Coordinate plot with axes:*
```typst
#cetz.canvas({
  import cetz.draw: *
  // Axes
  line((-0.5, 0), (6, 0), mark: (end: "stealth"), stroke: 1pt)
  line((0, -0.5), (0, 4), mark: (end: "stealth"), stroke: 1pt)
  content((6, -0.3), $x$)
  content((-0.3, 4), $y$)
  // Curve
  merge-path({
    for x in range(1, 51) {
      let xv = x / 10
      let yv = calc.sqrt(xv)
      line((xv, yv),)
    }
  }, stroke: positive + 1.5pt)
})
```

*CeTZ with Touying animation (using touying-reducer):*
```typst
#slide[
  #cetz-canvas({
    import cetz.draw: *
    rect((0, 0), (5, 5))
    (pause,)                // <-- tuple syntax, NOT #pause
    circle((2.5, 2.5), radius: 1)
    (pause,)
    line((0, 0), (5, 5), stroke: red)
  })
]
```

*Flowchart with Fletcher (preferred for node-edge diagrams):*
```typst
#fletcher-diagram(
  node-stroke: 0.5pt,
  node-corner-radius: 3pt,
  spacing: (15mm, 10mm),
  node((0, 0), [Input $x$]),
  node((1, 0), [Commit]),
  node((2, 0), [Prove]),
  node((3, 0), [Verify]),
  edge((0, 0), (1, 0), "->", label: [encode]),
  edge((1, 0), (2, 0), "->"),
  edge((2, 0), (3, 0), "->", label: [$pi$]),
)
```

*Annotated brace:*
```typst
#cetz.canvas({
  import cetz.draw: *
  // ... content ...
  // Use cetz.decorations for braces
  line((1, 0), (1, 3), stroke: (dash: "dashed", paint: neutral))
  content((1.3, 1.5), text(size: 0.8em)[annotation])
})
```

**CeTZ checklist:**
```
[ ] No label-label overlaps
[ ] No label-curve overlaps
[ ] Diagram fits within remaining slide space
[ ] ALL marked points computed from functions — no hardcoded y-values
[ ] Consistent visual semantics (solid/dashed, filled/hollow)
[ ] Arrow annotations: FROM label TO feature
[ ] Axes extend beyond all data points
[ ] Labels legible at presentation size
```

**CeTZ vs Fletcher decision guide:**
- **CeTZ**: coordinate plots, geometric constructions, custom diagrams, function graphs
- **Fletcher**: flowcharts, state machines, protocol diagrams, any node-edge graph
- When in doubt, use Fletcher for node-edge and CeTZ for everything else

### 2.7 `excellence [file]`

Comprehensive multi-dimensional review. **Use the Agent tool to dispatch review agents in parallel.**

**Parallel agent dispatch:**

1. **Visual audit agent** — overflow, font consistency, block fatigue, spacing, transitions
2. **Pedagogical review agent** — 13 patterns, deck-level checks
3. **Proofreading agent** — grammar, typos, citation consistency, notation
4. **CeTZ review agent** (if file contains CeTZ/Fletcher) — label overlaps, geometric accuracy, spacing
5. **Domain review agent** (optional) — substantive correctness

**After all agents return**, synthesize a combined report with quality score.

### 2.8 `visual-check [file]`

**PDF→image systematic visual review.** Converts compiled PDF to images, then reviews each slide visually. This catches issues invisible in source code — Typst's layout engine rarely warns about visual overflow.

**Why this matters:** Unlike LaTeX, Typst doesn't emit overfull hbox warnings. Content that exceeds the slide boundary is silently clipped. The only way to catch these issues is visual inspection.

**Workflow:**

1. **Compile** (if not already compiled):
   ```bash
   typst compile FILE.typ
   ```

2. **Convert PDF to images** using PyMuPDF:
   ```python
   import fitz
   doc = fitz.open('FILE.pdf')
   n = doc.page_count
   zoom = 200 / 72  # 200 DPI
   matrix = fitz.Matrix(zoom, zoom)
   for i in range(n):
       page = doc.load_page(i)
       pixmap = page.get_pixmap(matrix=matrix)
       pixmap.save(f'/tmp/slide-{i+1:03d}.jpg', output='jpeg')
   doc.close()
   print(f'Exported {n} slides')
   ```
   Or via bash: `python3 -c "import fitz; ..."`

   **Fallback** (if PyMuPDF unavailable): Use the Read tool directly on the PDF — Claude Code is multimodal and can read PDF files page by page with the `pages` parameter (max 20 pages per request).

3. **Systematic per-slide inspection** — use Read tool to view each image, checking:
   - [ ] No text overflow at any edge (top, bottom, left, right)
   - [ ] No content clipped by slide boundary (Typst silently clips, no warning)
   - [ ] All text legible (no text smaller than readable at presentation distance)
   - [ ] Tables and equations fit within slide width
   - [ ] CeTZ/Fletcher diagrams: labels not overlapping with lines, dots, or other labels
   - [ ] CeTZ computed points visually lie on their curves
   - [ ] Consistent font sizes across similar slide types
   - [ ] Adequate contrast between text and background
   - [ ] Colored blocks contain content within their borders (no visual spill)
   - [ ] `#pause`-based reveals look correct in handout mode (all content visible)
   - [ ] No visual clutter (too many elements competing for attention)

4. **Report** per issue:
   ```
   ### Slide N: [slide title]
   - **Issue:** [description]
   - **Severity:** Critical / Major / Minor
   - **Fix:** [specific recommendation]
   ```

**Tip:** For iterative editing, use `typst watch FILE.typ` to auto-recompile on save, then re-read the PDF periodically.

### 2.9 `validate [file] [duration]`

**Automated quantitative validation.**

1. **Compile and count pages:**
   ```bash
   typst compile FILE.typ && python3 -c "import fitz; print(fitz.open('FILE.pdf').page_count)"
   ```
   Compare against timing allocation table.

2. **Source code static checks:**
   - Count `#pause` usage — flag if excessive (>3 per slide average)
   - Check for missing bibliography/references section
   - Count slides with >2 styled blocks
   - Check all imports resolve

### 2.10 `extract-figures [pdf] [pages]`

**Extract figures from a paper PDF for inclusion in Typst slides.**

Workflow same as Beamer skill, but generate Typst snippets:

*Full-width figure:*
```typst
== Frame Title

#align(center)[
  #image("figures/fig-label.png", width: 85%)
]
#text(size: 0.8em)[Source: Author et al. (2024)]
```

*Figure + text side-by-side:*
```typst
#slide(composer: (1fr, 1fr))[
  #image("figures/fig-label.png", width: 100%)
  #text(size: 0.8em)[Source: Author et al.]
][
  Key observations:
  - Point 1
  - Point 2
]
```

---

## 3. VERIFICATION PROTOCOL

**Every task ends with verification.** Non-negotiable.

```
[ ] Compiled without errors (typst exit code 0)
[ ] No warnings in stderr
[ ] All imports resolve (packages auto-downloaded)
[ ] PDF opens and renders correctly
[ ] Visual spot-check of modified slides
```

---

## 4. DOMAIN REVIEW (Template)

Same 5 lenses as Beamer skill:

1. **Assumption stress test** — all assumptions stated, sufficient, necessary?
2. **Derivation verification** — each step follows?
3. **Citation fidelity** — slide accurately represents cited paper?
4. **Code-theory alignment** — code implements exact formula from slides?
5. **Backward logic check** — read conclusion→setup, every claim supported?

---

## 5. TROUBLESHOOTING

**Error:** PDF has way too many pages / slides look half-empty
**Cause:** `#pause` creates animation subslides — each becomes a separate PDF page. A slide with 4 pauses = 4 pages in the PDF, with the first 3 showing only partial content.
**Fix:** Add `config-common(handout: true)` to the theme `.with()` call. This collapses all subslides into one page with all content visible. Only disable for live presentation with pdfpc/pympress.

**Error:** Outline slide looks like a textbook table of contents (page numbers, dotted leaders, nested section numbers)
**Cause:** Using `#outline()` auto-generates a book-style TOC. Combined with `numbly` heading numbering, it produces "1.1 Title ......... 8" formatting.
**Fix:** Never use `#outline()` for slides. Write a manual bullet list instead. Never use `numbly` or `#set heading(numbering: ...)`.

**Error:** `#let myvar = $bb(G)$` then `$#myvar_1$` produces error or wrong output
**Cause:** Top-level `#let` assigns a content value; using `#` inside math mode to call it doesn't compose well with subscripts/superscripts.
**Fix:** Write the math directly: `$bb(G)_1$`. Or define math-safe functions: `#let GG(sub) = $bb(G)_#sub$` and call as `#GG[1]` outside math mode.

**Error:** `unknown import` or `package not found`
**Cause:** Wrong package version or name.
**Fix:** Check Typst Universe (https://typst.app/universe) for correct package name and latest version. Typst auto-downloads packages on first compile.

**Error:** `content does not fit` or visual overflow
**Cause:** Too much content for the slide.
**Fix priority:**
1. Split into two slides
2. Shorten text (telegraphic style)
3. Use `#utils.fit-to-height(1fr)[...]` or `#utils.fit-to-width(1fr)[...]` to auto-scale
4. Reduce font size with `#text(size: 0.9em)[...]` (last resort)

**Error:** Math rendering issues
**Cause:** Typst math syntax differs from LaTeX.
**Common conversions:**
- `\mathbb{F}` → `bb(F)`
- `\mathcal{O}` → `cal(O)`
- `\text{...}` inside math → `"text"` (quoted strings in math mode)
- `\left( \right)` → `lr(( ))` (use `lr()` for auto-sizing delimiters)
- `\sum_{i=0}^{n}` → `sum_(i=0)^n`
- `\frac{a}{b}` → `a / b` or `frac(a, b)`
- `\begin{align}` → use `$ ... \ ... $` with `\` for line breaks and `&` for alignment
- `\binom{n}{k}` → `binom(n, k)`
- `\langle, \rangle` → `angle.l, angle.r`
- `\| \|` → `norm(...)` or `lr(bar.double ... bar.double)`

**Error:** CeTZ diagram not rendering
**Cause:** Missing import or wrong canvas syntax.
**Fix:** Ensure both `#import "@preview/cetz:0.4.2"` at top level AND `import cetz.draw: *` inside `cetz.canvas({...})`.

**Error:** `#pause` not working inside CeTZ canvas
**Cause:** CeTZ uses a different pause mechanism.
**Fix:** Use `(pause,)` tuple syntax inside CeTZ canvases (not `#pause`). For Fletcher diagrams, use bare `pause`.

**Error:** `#pause` not working inside `context` expression
**Cause:** Touying's `#pause` cannot penetrate `context` boundaries.
**Fix:** Use callback-style: `#slide(repeat: N, self => [#let (uncover, only) = utils.methods(self); ...])`. Must manually set `repeat` count.

**Error:** Page layout broken / `set page(...)` has no effect
**Cause:** Touying overrides `set page()`.
**Fix:** Use `config-page(...)` inside the theme `.with()` call instead. E.g., `config-page(margin: (x: 2em, y: 1em))`.

**Error:** Theorems not numbered correctly with animations
**Cause:** Counter increments on each subslide.
**Fix:** Add `config-common(frozen-counters: (theorem-counter,))` to theme configuration.

**Error:** Bibliography not rendering
**Cause:** Missing `.bib` file or wrong path.
**Fix:** Ensure the `.bib` file exists and use `#bibliography("refs.bib")` at the end. Typst supports BibTeX and Hayagriva formats. Alternative: use `config-common(show-bibliography-as-footnote: bibliography("refs.bib"))` to show citations as footnotes instead of a bibliography page — then cite with `@key` in text.

**Error:** Touying slide not splitting on headings
**Cause:** Heading level mismatch with theme expectations.
**Fix:** Most Touying themes use `=` for sections and `==` for slides (slide-level=2). Exception: `dewdrop` uses slide-level=3 (`=` section, `==` subsection, `===` slide). For explicit control, use `#slide[...]`.

**Error:** `**bold**` renders as colored alert text instead of normal bold
**Cause:** Some themes enable `show-strong-with-alert: true` by default.
**Fix:** Add `config-common(show-strong-with-alert: false)` to the theme `.with()` call.

**Error:** Figure/theorem numbers jump during animation
**Cause:** Counters increment on each subslide.
**Fix:** Freeze the relevant counter: `config-common(frozen-counters: (theorem-counter,))` or `config-common(frozen-counters: (figure.where(kind: image),))`.

---

## 6. TYPST ↔ BEAMER MIGRATION CHEATSHEET

Quick reference for converting between the two systems:

| Beamer | Typst (Touying) |
|--------|----------------|
| `\begin{frame}{Title}` | `== Title` or `#slide[...]` |
| `\frametitle{T}` | `== T` (heading-based) |
| `\pause` | `#pause` |
| `\begin{itemize}` | `- item` (dash lists) |
| `\begin{enumerate}` | `+ item` (plus lists) |
| `\textbf{bold}` | `*bold*` |
| `\textit{italic}` | `_italic_` |
| `\texttt{code}` | `` `code` `` |
| `\begin{block}{T}` | `#block(title: [T])[...]` or theorem envs |
| `\begin{alertblock}{T}` | `#alert[...]` or `#block(fill: red.lighten(90%))[...]` |
| `\begin{columns}` | `#slide(composer: (1fr, 1fr))[...][...]` |
| `\includegraphics` | `#image("path", width: 80%)` |
| `\begin{tabular}` | `#table(columns: ..., ...)` |
| `\begin{tikzpicture}` | `#cetz.canvas({...})` |
| `\begin{align}` | `$ ... \ ... $` with `&` alignment |
| `\cite{key}` | `@key` (with bibliography) |
| `\footnote{text}` | `#footnote[text]` |
| `\hyperlink{label}{text}` | `#link(label("name"))[text]` |
| `xelatex` (3-pass) | `typst compile` (single pass, ~100ms) |

---
> Source: [Noi1r/typst-slide](https://github.com/Noi1r/typst-slide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
