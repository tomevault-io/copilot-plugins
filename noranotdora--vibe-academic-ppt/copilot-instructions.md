## vibe-academic-ppt

> You have access to several specific skills to assist in building the PPT. You **MUST** proactively use these skills whenever relevant:

# AGENTS: Build Academic Paper Presentation Slides with Vibe Coding

## 1. How to Use Built-in Skills (IMPORTANT)
You have access to several specific skills to assist in building the PPT. You **MUST** proactively use these skills whenever relevant:
- **`academic-ppt-planner`**: Invoke this when starting a new project, outlining slides, or refining presentation requirements based on materials in `reference/`.
- **`illustration-prompt-exporter`**: Invoke this when the deck needs standalone generated-image prompts, screenshot instructions, or a reusable visual prompt map.
- **`presentation-personalizer`**: Invoke this before generating a new PPT or script to retrieve, apply, or update specific presentation rules and preferences.
- **`tex-ppt-structuring`**: Invoke this when structuring the slides, integrating figures into TeX, or generating the final speech script.
- **`tex-builder`**: Invoke this when you encounter TeX compilation errors, or when you need guidance on building the PDF in the `output/` directory.

## 2. End-to-End Workflow
1. **Preparation**: Create a `reference/` folder for input materials (paper PDF, course requirements) and an `output/` folder for all generated code, figures, and PDFs. **MANDATORY**: Copy all contents from the provided `template/` directory (if any) into the `output/` folder to serve as the base for the presentation.
2. Ask the user to provide or approve a lightweight markdown outline before generating the full PPT. If the user only has rough notes, normalize them into a `.md` outline first.
3. Read the plan and invoke the `academic-ppt-planner` skill to complete requirement refinement based on materials in `reference/`.
4. Generate a page-level outline from refined requirements.
5. Generate a visual plan before TeX drafting: for each page, decide source figure / generated technical diagram / real screenshot / table, assign aspect ratio, and write image-generation prompts when needed. You may suggest nanobanana or other image-generation AI as the production tool, but keep prompts portable.
6. If the visual plan contains several generated diagrams or screenshots, invoke `illustration-prompt-exporter` and save a standalone prompt map.
7. Draft `pre.tex` content and `pre.bib` references from the outline into the `output/` folder by modifying the copied template files.
8. Invoke the `tex-ppt-structuring` skill to organize figures and place them by narrative logic (e.g., `output/fig/`).
9. Compile with XeLaTeX in the `output/` folder. If build fails, invoke the `tex-builder` skill.
10. Export representative screenshots and inspect layout geometry before finalizing.
11. Generate a final, directly deliverable speech script.

## 3. Mandatory First Step: Requirement Refinement
Before writing slides, always output:
- Presentation goal in one sentence
- Target audience (instructor / classmates / mixed)
- Duration and page constraints
- Core paper contributions (3–5 items)
- Must-cover figure list (key paper figures)
- Generated diagram / screenshot strategy
- Risk list (formula density, unreadable figures, page overflow)

## 4. Prompt Rules for the Agent
### 4.1 Writing Style Rules
- Use an academic presentation tone: objective, accurate, traceable.
- Prefer the structure “problem -> method -> result -> conclusion”.
- Keep each page to 3–6 points, one sentence per point.
- Keep formulas minimal and add one plain-language explanation.

### 4.2 Visual-Anchor Rules
- Each page should include at least one visual anchor (figure / flowchart / table).
- Prioritize original paper figures in their logical argument positions.
- After each key figure, provide a short “figure-speaking sentence”.
- When original figures are unavailable, use equivalent diagrams.
- For generated diagrams, write prompts in advance with a unified style instruction: clean academic vector technical diagram, light background, thin lines, rounded rectangles, blue/teal/gray palette with subtle warm accents, minimal labels or placeholder labels, no watermark, no logo.
- Prefer 3:4 diagrams for side-column visuals in the SYSU-style Beamer template. Use 16:9 only for dense overview, three-column comparison, swimlane workflow, or full-slide explanation.
- Use real screenshots for product interfaces, GitHub repositories, terminal results, real wiki pages, and demos where authenticity matters.
- Record the intended image/text ratio for every slide before placing figures.

### 4.3 Figure Placement Rules
- Extract section structure and figure IDs first (Figure 1, Figure 2, ...).
- Map figure positions in the storyline: background -> method -> experiments -> conclusion.
- Preserve mapping for key figure IDs to support verbal references.
- Avoid figure stacking: at most 2 major figures per page.
- For wide figures, prefer full-width top-bottom layout or full-slide visual treatment over cramped side columns.
- For portrait figures, do not blindly widen the image column if `keepaspectratio` makes the image height-limited.
- If text wraps into isolated words or the final line is much shorter than the rest, widen the text column or simplify the sentence before shrinking font size.
- Reduce block wrappers on image-led slides when their title bars create unnecessary vertical gaps.

## 5. Recommended Execution Template
- Inputs: files in `reference/` (paper PDF, course requirements), target duration, template directory
- Outputs: files generated in `output/`:
  - Page-level outline (page title + speaking goal)
  - `pre.tex` draft and `pre.bib`
  - Figure resource plan (filename, source, target page)
  - Build commands and issue-fix log
  - Final `pre.pdf` presentation

## 6. Quality Gate (Definition of Done)
- `pre.pdf` compiles reliably.
- Page structure is complete and duration-fit.
- Figures and conclusions are clearly paired.
- References are traceable and citation build is clean.

---
> Source: [NoraNotDora/vibe-academic-ppt](https://github.com/NoraNotDora/vibe-academic-ppt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
