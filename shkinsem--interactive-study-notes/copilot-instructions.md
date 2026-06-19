## interactive-study-notes

> Use when a user wants to turn lecture slides, exercises, and past papers into interactive HTML study notes for exam prep. Triggers on phrases like "速通笔记", "交互式 html 笔记", "把课件做成网页", "把我当三岁小孩讲", "exam crash course", "interactive study notes", "lecture-to-html", "study guide from PDFs", or any combination of (a course folder / lecture materials) + (review / exam / cram / notes intent). Also fires on improvement requests for existing interactive notes ("improve these notes", "expand the cheatsheet", "the layout is too sparse"). Covers single-lecture pages, multi-lecture courses with a hub index, offline-capable single-file delivery, and companion artifacts like A4 cheat-sheet PDFs.


# Interactive Study Notes

This skill converts a folder of lecture materials (PDFs, slides, past papers, homework) into a set of polished, interactive HTML study notes — the kind you'd actually want to cram from the night before an exam.

The output is **offline-capable plain HTML/CSS/JS** — no build step, no framework, double-clickable, OneDrive-friendly. Math renders via KaTeX. Interactive components include flip-card term drills, MCQ self-quizzes with instant feedback, step-by-step algorithm animations, sidebar navigation with scroll progress, and per-lecture cheat sheets.

## When to trigger

Pattern-match these (mix freely across languages):

- "速通笔记" / "复习笔记" / "考前笔记" / "exam crash course" / "study guide" / "cheat sheet"
- "期末降至 / 死到临头" + any course materials
- "把 [PDF/PPT/课件/lectures] 做成 html / 网页 / 交互式 / interactive"
- "把我当三岁小孩教" / "ELI5" / "深入浅出" / "explain like I'm five"
- Any mention of converting lecture slides into web-viewable study materials
- Improvement requests on existing interactive notes ("看一下当前工作区的交互笔记，然后予以改进", "the notes are too sparse", "add more practice questions")

When triggered, **read this whole file plus `references/workflow.md`**, then proceed. The skill is a workflow, not a one-shot generator.

## The product shape (non-negotiable)

These constraints come from real student feedback and have been validated across multiple courses:

1. **Per-lecture HTML files + one `index.html` hub** in an `interactive note/` (or similarly named) folder. One file per chapter, not a monolithic single-page app.
2. **Offline-capable**: works by double-clicking the file (`file://` protocol). No build step, no npm, no CDN dependencies for *core* functionality. CDN is allowed only for KaTeX/MathJax (math rendering) — the page must still display readable text if offline.
3. **Shared CSS/JS** when producing ≥3 lectures: copy this skill's `assets/notes-<skin>.css` + `assets/notes-<skin>.js` into `interactive note/assets/` and reference via relative path. Don't reinvent the components — copy and adapt.
4. **Pedagogical model**: ELI5-first (everyday analogy before formal definition) + exam-focused (every concept ends with self-quiz) + **zero prior knowledge** (write for a reader who has NEVER opened the lecture slides — not for someone refreshing what they already saw. Every algorithm name, abbreviation, metric, data structure, and math symbol must be expanded inline on first appearance. No undefined-term chains. Past complaint: "假设我看过课件了，但是我希望是把我当作没看过课件的人来教"). Every concept card follows a fixed 6-section structure (see below).
5. **HOMEWORK + PAST PAPERS are MANDATORY inputs**. If the workspace has a `hw/`, `homework/`, `assignment/`, `past paper/`, `previous exam/`, or similar folder, you MUST read every file and mine for: (a) recurring question types, (b) worked examples to drop into 🌰, (c) traps to drop into ⚠️, (d) MCQs/short-answer items to seed 🎯 final quiz. Failing to do this produces "too abstract" notes — the textbook explains *concepts*, the homework reveals *what gets tested*.
6. **"Dark-horse questions" (🌑 暗箭考点)**: After mapping past-paper coverage, explicitly identify slide content that appears in the lecture but is **absent from past papers/homework/exercises**. These untested-but-in-scope topics are the **most likely surprise questions** — they haven't been "burned through" yet. The 🎯 final quiz of each lecture MUST include 2-3 questions on these dark-horse topics, not only past-paper paraphrases.

## Canonical concept card (6 sections)

Every concept card in a lecture page follows this scaffold. The emojis are part of the brand — keep them as section markers (they make scanning the page much faster):

1. **🍼 ELI5 / 大白话** — Plain-language analogy first. No jargon. Like explaining to a curious 5-year-old.
2. **📖 Formal definition / 正式定义** — Formal definition with technical terms preserved in their original language (e.g., keep "Apriori", "support", "lift ratio" in English even when surrounding prose is in another language — exams usually use the canonical English term).
3. **🌰 Example / 例子** — A concrete worked example with real numbers, or a famous case study. For math/algorithm courses: actual numerical walk-through. For concept courses: a famous case (e.g., Kodak/Blockbuster, Netflix, etc.).
4. **⚠️ Exam trap / 考点陷阱** — The subtle distinction the student is likely to get wrong. Sourced from past papers when available; from common misconceptions otherwise.
5. **🔄 Flip-cards / 翻卡** — 2-4 key terms as 3D flip-cards (click to flip). Front = term, back = definition. Drills vocabulary.
6. **❓ Self-quiz / 自测** — 1-3 MCQs with **instant feedback**: click an option → immediately get green/red + reveal explanation. No "submit at end" button.

At the END of each lecture page, two more sections:

- **📌 Cheat-sheet / 一页速记** — Bullet-point cram sheet of everything on the page (10-20 dense bullets).
- **🎯 Final quiz / 综合自测** — 3-6 MCQs covering the whole lecture, plus a **🌑 Dark-horse questions / 暗箭考点** subsection with 2-3 MCQs on slide-only topics.

## Required interactive components

These are validated, copy-from-the-skill components. Don't reinvent. See `references/components.md` for full HTML/CSS/JS snippets.

| Component | When to use | Critical detail |
|---|---|---|
| **Flip-cards** | Terminology drilling | 3D CSS flip (`transform: rotateY`), click-toggle, NOT hover (mobile friendliness) |
| **MCQ self-quiz** | Per-card + lecture-end | Click an option → instant color (green/red) + reveal explanation panel. Not "submit at end" |
| **Sidebar nav + progress bar** | Long pages (>4 sections) | Sticky on desktop, collapses to top on mobile. Highlights current section on scroll |
| **Collapsible sections** | Long derivations / proofs | `<details>` is fine for simple cases |
| **Step-by-step animated diagrams** | Algorithm walkthroughs (clustering iterations, tree construction, ranking iterations) | Buttons ①②③④ that progressively reveal stages. Use SVG with `<g class="step" data-step="K">` + step-controls — see components.md |
| **LaTeX rendering** | Math-heavy courses | KaTeX from CDN with `defer`. The math falls back to source text if offline |
| **Cheat-sheet box** | End of page | Tight, dense, scannable summary — print-friendly CSS included |

## Visual design: pick a skin

This skill ships with THREE validated skins. **Pick one based on the subject matter — do not blindly default to one.**

### Skin A — "Bright Friendly" (`assets/notes-bright.css` + `notes-bright.js`)

- **Best for**: Business / innovation / humanities / pedagogy-heavy courses. Workshops, storybook explanations, casual tone.
- **Palette**: warm beige/cream background (#FBF7F0), deep slate text, warm orange (#E76F51) primary accent, sage green secondary.
- **Type**: system stack (`system-ui, -apple-system, "Segoe UI", "PingFang SC", ...`).
- **Vibe**: classroom storybook, encouraging, generous whitespace, rounded corners.
- **Class contract**: standard (see below)

### Skin B — "Academic Terminal" (`assets/notes-terminal.css` + `notes-terminal.js`)

- **Best for**: CS / data / engineering / math / physics / hard-science courses. Algorithms, formulas, technical depth.
- **Palette**: deep ink background (#0B0F14), warm parchment text (#E8E0D0), electric lime (#D4F227) primary accent, cyan for math/definitions, coral for traps, gold for ELI5.
- **Type**: Fraunces (display italic serif) + Newsreader (body serif) + JetBrains Mono (technical), all from Google Fonts CDN.
- **Vibe**: dark scholarly lab, data-mining-research feel, ASCII micro-flourishes (`▌`, `>>`, `// METHOD`), 2-column layout on wide screens to maximize density. Sharp corners.
- **Class contract**: standard (see below)

### Skin C — "Paper Editorial" (`assets/notes-editorial.css` + `notes-editorial.js`)

- **Best for**: Operations / supply chain / case-study / quantitative-business / capstone-style courses. Magazine-feature feel.
- **Palette**: cream-paper background (#FAF6EF), dark ink text (#1A1A2E), per-section accent colors (forecast green / inventory orange / revenue maroon / scm navy — adapt to your course's chapter palette), yellow highlight (#FFE89C) for offset-shadow boxes.
- **Type**: Fraunces (bold + italic display) + Newsreader (body) + JetBrains Mono (technical) — same family as Skin B but used very differently. Massive italic display headlines with red emphasis-em.
- **Vibe**: print magazine / editorial. SVG noise overlay for paper texture. Double-rule section dividers. Bold dark "analogy-box" with offset yellow shadow. Per-chapter color coding via `data-c` attribute on `.page-header`.
- **Class contract**: **DIFFERENT** — uses `.wrapper`, `.page-header[data-c]`, `.analogy-box`, `.callout-box`, etc. instead of the standard `.sidebar / .concept-card / .card-section.eli5/...`. NOT plug-compatible with Skin A/B — if you use this skin, the HTML structure of each lecture must follow its conventions. See the inline comments in `assets/notes-editorial.css` for the full class list.

### Choosing heuristic

- Course code matches `COMP / ELEC / MATH / PHYS / STAT / CS / EE / ME / ChE` → Skin B (terminal)
- Course code matches `ACCT / FINA / MGMT / ECON / MKTG / SOSC / HUMA / ARTS` → Skin A (bright)
- Course code matches `ISOM / OPER / SCM / LGT / case-study-heavy` → Skin C (editorial)
- Ambiguous → **ask**: "走暖色 storybook、深色 terminal，还是杂志 editorial？" / "Bright storybook, dark terminal, or magazine editorial skin?"
- **Never default silently** — wrong skin is a common complaint. If the user already has a skin established for another course in the same workspace, match it.

### Class contract (Skin A + B share this; Skin C does NOT)

- Layout: `.sidebar`, `.progress-bar`, `.progress-fill`, `main`, `.hero`, `.tagline`
- Concept card: `.concept-card`, `.concept-title`, `.card-section.eli5/.formal/.example/.trap/.flipcards/.quiz`
- Flip-cards: `.flipcard-grid`, `.flipcard`, `.flipcard-inner`, `.flipcard-front`, `.flipcard-back`, `.flipcard.flipped`
- MCQ: `.mcq[data-correct]`, `.mcq-question`, `.mcq-options`, `.mcq-options button[data-option]`, `.mcq-explanation`, `.correct`, `.incorrect`
- Stepped diagrams: `.stepped-diagram[data-steps]`, `.diagram-svg`, `.step[data-step]`, `.step.visible`, `.step-controls button[data-step-btn]`, `.step-narration`
- Cheatsheet / quiz: `.cheatsheet`, `.cheat-bullets`, `.final-quiz`
- Hub (index): `.hub`, `.hub-hero`, `.hub-grid`, `.hub-card.priority-high/-medium`, `.hub-num`, `.hub-title`, `.hub-meta`
- Badges: `.pill`, `.pill-must`, `.pill-hot`, `.pill-easy`

## The workflow (4 phases)

Read `references/workflow.md` for the full step-by-step. Quick summary:

### Phase 1 — Scope discovery (DON'T SKIP)
1. Find the exam-scope document: usually named `Final Guide.md`, `topics and final exam.pdf`, `syllabus.pdf`, or similar. **Read it first.** If absent, ask the user what's in scope.
2. List all lecture files (PPT/PPTX/PDF). Convert PPT→PDF if needed (PowerPoint COM on Windows, or LibreOffice headless on Linux/Mac).
3. **Read homework + exercises + past papers in full.** Maintain a cross-reference table: topic → past-paper hits → HW hits → question types. This table is the BACKBONE of exam-focused notes — without it, you're just rephrasing slides.
4. Confirm scope with the user before producing — output a prioritized list of lectures with exam-priority badges (⭐⭐⭐ / ⭐⭐ / ⭐).

### Phase 2 — Template (build ONE, validate, then batch)
5. Pick the highest-priority lecture as the template. Ideally one that exercises many feature types (math + diagram + concept + worked example).
6. Build the template as a self-contained HTML first (CSS/JS inlined). Use the chosen skin's CSS as the base. Verify it: if a headless browser is available, screenshot the rendered page (and script a click/drag for the interactive bits); otherwise show the user and ask them to click through.
7. Once approved, copy `assets/notes-<skin>.css` and `assets/notes-<skin>.js` to the output folder. Rename them to `assets/notes.css` and `assets/notes.js` (so subsequent HTML files just reference these). Refactor the template to use the shared files.

### Phase 3 — Batch production
8. For each remaining lecture: dispatch a content-extraction subagent (one per lecture, or batched 2-3 per agent if scope is large). Each subagent **reads the lecture PDF + corresponding exercises + relevant past-paper questions** and writes a structured Markdown content outline (NOT HTML) to a scratch folder.
9. Then dispatch HTML-rendering subagents that read each Markdown outline + the template HTML and produce the final per-lecture HTML. Or do this inline if scope is small (<5 lectures).
10. Build `index.html` hub: card grid of all lectures, sorted by exam priority, with difficulty stars / past-paper indicators / estimated read time.

### Phase 4 — Polish
11. **Add 🌑 dark-horse MCQs** (one polish-pass agent per lecture, or inline). Re-read the lecture PDF for each lecture, identify slide content NOT covered by past papers/homework, and add 2-3 untested-topic MCQs to that lecture's final quiz section.
12. Optional: companion A4 cheat-sheet PDF for math-heavy courses.
13. Verify the output. If a headless browser is available, screenshot a couple of finished pages (and drive one or two interactions via a Playwright/CDP script) to confirm rendering + behavior; in any case, ask the user for a final spot-check, since a human click-through is the fastest confidence check.

## Common iteration patterns

After the user reviews, expect these requests:

- **"内容偏稀疏 / too sparse"** → expand 🌰 examples and ⚠️ traps; add more flipcards; check that homework worked examples are integrated
- **"右边大片空白 / too much empty space"** → on Skin B, the 2-col layout for wide viewports usually fixes this; on Skin A, increase `max-width` of `main`
- **"样式不统一 / inconsistent style"** → ensure all lectures reference the same `assets/notes.css` (not inline styles)
- **"加题目 / more practice questions"** → expand 🎯 final quiz, mine more past-paper questions
- **"做个 cheat sheet PDF"** → see Phase 4.2; use `weasyprint` or `reportlab` with the lecture's 📌 sections

Handle each as a focused edit, NOT a full rebuild. The shared-assets pattern means most style changes are one-file edits.

## Pitfalls (each was a real complaint at least once)

- **❌ Just rephrasing slide headings** — every concept card needs a worked example AND an exam trap. If you can't find an example, READ the exercise PDF. The course materials are usually rich; the slide is just the index.
- **❌ Mocking math/algorithms** — numerical correctness matters. Verify K-means iterations, decision-tree splits, Bayes posteriors, etc. with actual computation. If you wrote "0.42", it had better compute to 0.42.
- **❌ Single monolithic `index.html`** — with >3 lectures, split into per-lecture pages with a hub. Past feedback: "你其实可以分一下资产或者页面来设计，不用全部堆在一个 index 里面".
- **❌ Forgetting the ELI5 tone** — if a concept card opens "Concept X is a theory whereby...", you've failed. Start with "Imagine you..." or "想象一下你..."
- **❌ Assuming the reader has seen the slides** — every jargon term (algorithm names, abbreviations, metrics, math symbols) MUST be expanded inline on first appearance in every section. Past complaint: "假设我看过课件了，但是我希望是把我当作没看过课件的人来教".
- **❌ Defaulting silently to one skin** — match skin to subject. Past complaint when wrong: "怎么是 [other course] 那个设计风格".
- **❌ Two-column 50/50 layouts where one side is sparse** — leaves big empty rectangles. Skin B uses `flex-wrap` with `flex: 1 1 calc(50% - 9px)` so single sections expand. Skin A defaults to single column.
- **❌ Hardcoded light-theme inline backgrounds on a dark skin** — subagents love to drop inline styles like `style="background:#FFF6E8"` / `#FFFEFA` / `white` on `<tr>`, `<details>`, `<blockquote>`, callout `<p>`s (highlighting an answer row, an info box, etc.). On a DARK skin (Skin B) this paints a light box, but the text inside still inherits the light parchment `--text` color → **light-on-light, looks exactly like overlapping/garbled fonts**. This was a real bug (PCA & KNN "FORMAL GOAL" boxes, NN/decision-tree highlight rows). Prevention: (1) instruct rendering subagents to NEVER hardcode hex backgrounds — use theme variables (`var(--surface-2)`, `var(--eli5-bg)`, etc.) or semantic classes; (2) after generation on a dark skin, grep every produced HTML for `background:\s*#(FFF|fff|FAF|FBF|EDF|FDF|FFE)` and `background:\s*white` and replace with theme vars. The same applies in reverse if you ever hardcode dark backgrounds on a light skin. Bottom line: **inline colors are skin-coupled; keep all color in the shared CSS.**
- **❌ Claiming the interactivity works without checking** — If a headless browser is available (Chrome/Chromium + shell), DO verify: screenshot the rendered page for layout/fonts/math, and for interaction-heavy pages write a short Playwright/CDP script that clicks a flip-card / answers an MCQ / drags a demo point and screenshots the result. If no browser is available, say so honestly and defer to the user with specifics like "Open the file, click any 翻卡 in section 2 — should flip; answer a wrong MCQ option — should turn red with explanation visible." Never assert "it works" from reading the source alone.
- **❌ Building before reading scope** — wastes tokens producing notes for topics that aren't on the exam.
- **❌ Skipping past papers / homework** — produces "too abstract" notes. These materials reveal *what gets tested* in a way slides never do.
- **❌ Heavy frameworks** — no React, no Vite, no npm. Vanilla HTML/CSS/JS. The user double-clicks the file.

## Token efficiency

- **Subagent parallelism** is strongly recommended for batch production (Phase 3). Each subagent handles 1-2 lectures end-to-end.
- **Read summarized materials over raw PDFs** when available. If the workspace has `01_Concepts_Deep.md` or similar pre-digested files, prefer those.
- **Content-first, code-second**: have subagents extract structured content (Markdown outline) first, then a second pass renders HTML. This separates concerns and lets you batch-edit content without re-rendering HTML.
- **Use Haiku for static markup, Sonnet/Opus for content reasoning** — only for very large batches and only if user explicitly opts in (quality drops on tricky interactive components).

## Reference files in this skill

- `references/workflow.md` — Phase-by-phase workflow checklist with concrete commands and prompt templates
- `references/components.md` — Copy-paste-ready HTML/CSS/JS for every interactive component
- `assets/notes-bright.css` + `notes-bright.js` — Skin A baseline (Bright Friendly)
- `assets/notes-terminal.css` + `notes-terminal.js` — Skin B baseline (Academic Terminal)
- `assets/notes-editorial.css` + `notes-editorial.js` — Skin C baseline (Paper Editorial) — uses its own class contract, see CSS comments

## Closing the loop

After producing the notes, ALWAYS:

1. Tell the user the exact path to open (e.g., `interactive note/index.html`) — link with markdown URL syntax if in a chat UI.
2. Name 2-3 specific interactions for them to verify (e.g., "Click lecture-07 section 2's flip-card; answer the wrong option of any MCQ — should highlight red + show explanation").
3. If exam is imminent, end with low-key encouragement ("考试加油" / "good luck").

---
> Source: [SHKinsem/Interactive-Study-Notes](https://github.com/SHKinsem/Interactive-Study-Notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
