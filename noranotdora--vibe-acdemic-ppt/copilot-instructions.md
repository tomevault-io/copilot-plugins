## vibe-acdemic-ppt

> You have access to several specific skills to assist in building the PPT. You **MUST** proactively use these skills whenever relevant:

# AGENTS: Build Academic Paper Presentation Slides with Vibe Coding

## 1. How to Use Built-in Skills (IMPORTANT)
You have access to several specific skills to assist in building the PPT. You **MUST** proactively use these skills whenever relevant:
- **`academic-ppt-planner`**: Invoke this when starting a new project, outlining slides, or refining presentation requirements based on materials in `reference/`.
- **`presentation-personalizer`**: Invoke this before generating a new PPT or script to retrieve, apply, or update specific presentation rules and preferences.
- **`tex-ppt-structuring`**: Invoke this when structuring the slides, integrating figures into TeX, or generating the final speech script.
- **`tex-builder`**: Invoke this when you encounter TeX compilation errors, or when you need guidance on building the PDF in the `output/` directory.

## 2. End-to-End Workflow
1. **Preparation**: Create a `reference/` folder for input materials (paper PDF, course requirements) and an `output/` folder for all generated code, figures, and PDFs. **MANDATORY**: Copy all contents from the provided `template/` directory (if any) into the `output/` folder to serve as the base for the presentation.
2. Read the plan and invoke the `academic-ppt-planner` skill to complete requirement refinement based on materials in `reference/`.
3. Generate a page-level outline from refined requirements.
4. Draft `pre.tex` content and `pre.bib` references from the outline into the `output/` folder by modifying the copied template files.
5. Invoke the `tex-ppt-structuring` skill to organize figures and place them by narrative logic (e.g., `output/fig/`).
6. Compile with XeLaTeX in the `output/` folder. If build fails, invoke the `tex-builder` skill.
7. Generate a final, directly deliverable speech script.

## 3. Mandatory First Step: Requirement Refinement
Before writing slides, always output:
- Presentation goal in one sentence
- Target audience (instructor / classmates / mixed)
- Duration and page constraints
- Core paper contributions (3–5 items)
- Must-cover figure list (key paper figures)
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

### 4.3 Figure Placement Rules
- Extract section structure and figure IDs first (Figure 1, Figure 2, ...).
- Map figure positions in the storyline: background -> method -> experiments -> conclusion.
- Preserve mapping for key figure IDs to support verbal references.
- Avoid figure stacking: at most 2 major figures per page.

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
> Source: [NoraNotDora/vibe-acdemic-ppt](https://github.com/NoraNotDora/vibe-acdemic-ppt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
