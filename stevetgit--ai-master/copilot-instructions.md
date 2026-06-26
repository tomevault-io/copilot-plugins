## ai-master

> Instructions for Claude Code when working in this repo. This is AI-editing guidance, not human onboarding — see `docs/style-guide.md` for the human-readable equivalent.

# CLAUDE.md — AI Master Notes Repository

Instructions for Claude Code when working in this repo. This is AI-editing guidance, not human onboarding — see `docs/style-guide.md` for the human-readable equivalent.

---

## Repository layout

```
AI_Master/
├── templates/
│   ├── summary_template.tex   ← source of truth for summary.tex files
│   └── exams_template.tex     ← source of truth for exams.tex files
└── courses/
    └── <course-slug>/
        ├── summary.tex
        ├── exams.tex
        └── figures/
```

Course folders use lowercase kebab-case slugs (`reinforcement-learning`, `computer-vision`).

**What gets committed:** `.gitignore` whitelists course folders — only
`README.md`, `summary.{tex,pdf}`, `exams.{tex,pdf}`, `exams_solutions.{tex,pdf}`,
and `figures/` are tracked (plus the Deep RL `cheatsheet.{tex,pdf}` exception).
Everything else stays local: lecture slides (`slides/`), raw exam scans,
exercise notebooks, quiz/real-exam dumps, `.claude/`, build artifacts. To
publish a new document type, add an explicit `!` rule to the courses block
in `.gitignore`.

---

## summary.tex conventions

- **Structure:** `\chapter` per major topic, `\section` / `\subsection` within.
- **Accent color:** `#1B3A5C` (deep navy) — never change this.
- **Philosophy:** running prose is the default; boxes are landmarks, not
  the primary content carrier. When in doubt, write prose.

### Box types (summary.tex)

| Box | Color | Use for |
|-----|-------|---------|
| `\begin{defbox}[title={…}]` | Blue (#1A5276) | Formal definitions |
| `\begin{thmbox}[title={…}]` | Red (#922B21) | Key theorems, important formulas |
| `\begin{exbox}[title={…}]` | Green (#1E8449) | Worked examples |
| `\begin{intbox}` | Amber (#9A6A00) | Intuition, mental models |
| `\begin{notebox}[title={…}]` | Purple (#5B2C6F) | Exam traps, caveats |

Do NOT add boxes without a clear reason. Every unnecessary box dilutes
the signalling value of all boxes.

---

## exams.tex conventions

- **Structure:** `\section` per thematic topic (no chapters — article class).
  Sections should map to the chapter breakdown of the corresponding `summary.tex`.
- **Questions are NOT numbered manually** — `\questiontitle` auto-increments
  a counter.
- **Exact duplicates are removed.** When the same question appeared on
  multiple exams, keep one instance and add `\repeatnote` at the top.
- **Question wording is paraphrased, never copied verbatim.** Reword every
  question stem so it is not a literal copy of the original exam, while
  preserving the exact meaning and difficulty. Example: *"Which statement
  about molecular graphs and molecular fingerprints is correct?"* →
  *"Which of the following statements accurately describes the relationship
  between molecular graphs and molecular fingerprints?"* This is rephrasing
  for legal safety, not a content change.
- **Answer choices, numbers, and math are kept as-is.** Do not alter answer
  options, numeric values, or mathematical expressions when paraphrasing the
  stem; only fix wording where a scan was messy. The same applies to
  `exams_solutions.tex`, which shares the questions.

### Box types (exams.tex)

| Macro / Box | Color | Use for |
|-------------|-------|---------|
| `\questiontitle{Title}` … `\closequestion` | Blue (#1A5276) | Every exam question |
| `\begin{keybox}[title={…}]` | Green (#1E8449) | Key insight or answer hint |
| `\begin{notebox}[title={…}]` | Amber (#9A6A00) | Organizational notes only |

### Structural macros

| Macro | Effect |
|-------|--------|
| `\questiontitle{Short title}` | Opens a numbered blue question box |
| `\closequestion` | Closes the question box |
| `\repeatnote` | Prints italic "Asked multiple times." |
| `\choice` | `☐` bullet item for multiple-choice |
| `\answerline{3cm}` | Fill-in underline of specified width |
| `\workingspace{4cm}` | Dashed-border blank box for student work |

### Question types

| Question type | Format inside `\questiontitle` … `\closequestion` |
|---------------|--------------------------------------------------|
| **Multiple choice** | `\begin{itemize}[leftmargin=1.8em]` with `\choice` items |
| **Single number / short expression** | `\answerline{3cm}` inline at the end of the question text |
| **Calculation / short derivation** | `\workingspace{4cm}` on its own line after the question |
| **Open / long answer** | `\workingspace{7cm}` on its own line after the question |

The heights for `\workingspace` are guidelines — adjust to fit the expected length of the answer.

### Answer key

Every `exams.tex` file ends with an answer key:

```latex
\newpage
\section*{Answer Key}
\begin{enumerate}[label=\textbf{Q\arabic*.}, leftmargin=3em, itemsep=4pt]
  \item \textbf{B}
  \item $x = 0.5$
  \item [key steps: …]
\end{enumerate}
```

- The list is numbered manually and must match the auto-incremented question counter order.
- Keep answers concise: letter for MCQ, expression for fill-in, key steps/formula for derivations, bullet points for open questions.
- **Never** put answer content inside `\questiontitle` / `\closequestion` blocks — the question boxes must stay blank.

### exams_solutions.tex (worked solutions, optional)

`exams_solutions.tex` is the *solutions variant* of `exams.tex`: the same
questions in the same order, but instead of leaving the question boxes blank
and collecting answers in an Answer Key at the end, each question is followed
immediately by a green answer box.

- **Multiple choice:** mark correct options with a green check and incorrect
  ones with a red cross, each with a one-line justification (including why
  tricky near-miss distractors are wrong). Wild distractors that only exist
  to catch guessers need no explanation.
- **Calculation / derivation:** show the working.
- Keep `exams.tex` as the blank practice set with its Answer Key — do **not**
  merge the two files.
- Derived from `exams.tex` (no separate template). When `exams.tex` changes,
  update `exams_solutions.tex` to match.

### Math macros included in exams.tex

`\R`, `\E`, `\Prob`, `\KL{q}{p}`, `\dd` — add from `templates/summary_template.tex` as needed.

---

## Shared design rules (both files)

- Same `accent` color: `#1B3A5C`.
- Same `tcbset` global box style: `arc=2.5mm`, `boxrule=0.5pt`, thick left
  accent bar (`borderline west={3.5pt}{0pt}{…}`), `sharp corners=west`,
  `parbox=false`, `breakable`.
- Same typography: `lmodern`, `microtype`, `inconsolata`, `\parindent=0pt`,
  `\parskip≈0.6em`.
- Same title page structure: thick accent rule → large bold title → thin
  accent rule → author → date.

---

## Starting a new course document

```bash
# summary
cp templates/summary_template.tex courses/<slug>/summary.tex

# exams
cp templates/exams_template.tex courses/<slug>/exams.tex
```

Set `\coursetitle` in every document. Set `\basedon` in `summary.tex` when the
summary template provides it. Figure paths are relative: `figures/my-fig.png`.
For exam screenshots or figure-based questions, prefer `\examfigure{figures/...}`
over replacing the figure with a long prose description.

---

## Compiling

```bash
# Inside the course folder:
latexmk -pdf summary.tex
latexmk -pdf exams.tex
latexmk -C          # clean build artifacts
```

Always commit both `.tex` and `.pdf`.

---

## What NOT to do

- Do not edit `templates/summary_template.tex` or `templates/exams_template.tex` in-place — copy first.
- In `exams.tex`, do not add answers inside `\questiontitle` / `\closequestion`
  blocks — solutions belong only in the `\section*{Answer Key}` section at the end.
- Do not change accent colors or box color definitions per-document.
- Do not add a chapter level to `exams.tex` — it is article class.
- Do not copy exam-question wording verbatim — paraphrase every question stem (keeping the answer options, numbers, and meaning intact).
- Do not try to commit local study material (lecture slides, raw exam scans, exercise notebooks, quiz/real-exam dumps, cheat-sheet underlays). The `.gitignore` whitelist tracks only `summary`, `exams`, `exams_solutions`, `figures/`, and `README.md` per course (plus the Deep RL `cheatsheet`).

---
> Source: [steveTgit/AI_Master](https://github.com/steveTgit/AI_Master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
