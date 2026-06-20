## econ-review-master

> Use when the user wants to turn economics PDF lecture slides or an economics course folder into portable Markdown revision notes, a problem-first 21-hour review roadmap, a refined existing review outline, or a current-scope past exam to lecture map. Trigger for: 总结课件, 复习资料, 考点总结, cheat sheet, 公式速查, summarize slides, review notes, lecture summary, review roadmap, 复习路线图, 复习排期, 21小时复习计划, problem-first revision plan, 润色大纲, 修改复习大纲, current-scope past exam map, past exam to lecture mapping, 今年考纲真题映射, 真题对应课件页, 判断真题是否超纲, or requests to integrate economics lectures with problem sets, solutions, seminars, past exams, a syllabus, or an existing outline. The skill is folder-aware: scan the course directory before summarising, classify available materials, and produce a saved Markdown artifact by default. Do not use for homework solving, essay writing, non-economics materials, or PowerPoint files that have not been exported to PDF.


# Econ Slides Review

Turn economics course materials into concise, exam-oriented portable Markdown notes. Optimize for three things: economic intuition, source traceability, and integration with problem patterns. Do not merely compress slides.

## Operating Rules

- Use a PDF-capable intake method for all PDFs. If a `pdf` or `pdf-reading` companion skill is available, use it for extraction and page handling.
- Default deliverable: write one `.md` file to disk in the course directory. Do not stop with a chat-only summary unless the user explicitly asks for chat-only output.
- Default language: Chinese-first, with important economics terms preserved in English parentheses, such as `边际替代率 (Marginal Rate of Substitution, MRS)`.
- Default scope: if the user points to a folder, scan the folder before summarising. If the user points to one specific PDF, treat it as the lecture source and produce a lecture-only review.
- Required citations: every knowledge point, formula, definition, model result, technique, and exam observation must include a precise source locator wrapped in Markdown inline-code backticks, such as `` `Lec 3 课件页 12-18` ``.
- Required intuition: every formal result, definition, formula, theorem, or model conclusion must be followed by a `#### Economic Intuition` subsection. If the intuition is unclear, write `#### Economic Intuition` followed by `直觉待补充` instead of silently omitting it.
- Never fabricate problem recommendations. Add `｜推荐练习：...` inside the same inline-code source tag only when a matching problem set, exam, seminar, or solution item was actually found.
- Do not solve homework or write essays. For problem sets and exams, extract tested concepts, question patterns, and answer techniques only.

## Mode Selection

Infer the mode before starting deep reading. Respect scope-limiting phrases such as "只生成", "只润色", "不要完整笔记", "roadmap-only", or "outline refinement".

| Mode | Trigger | Output |
|---|---|---|
| Full review | Default for summary/review-note requests | `<stem>_review.md` with integrated notes, past exam analysis when available, 21-hour roadmap, and quick reference |
| Roadmap only | `只生成 roadmap`, `复习排期`, `21 小时`, `problem-first roadmap`, `从题目出发安排复习` | `<stem>_roadmap.md`; do not generate full notes |
| Current-scope past exam map | `今年考纲真题映射`, `真题对应课件页`, `判断真题是否超纲`, `current-scope past exam map`, `past exam to lecture mapping` | `<stem>_current_scope_exam_map.md`; map every past exam subquestion to current scope, lecture location, and related problem sets |
| Outline refinement | `润色大纲`, `修改复习大纲`, `review outline`, `outline refinement`, or user points to an existing outline | Refined outline saved as `<outline-stem>_refined.md` unless the user asks to overwrite |

Mode rules:

- In full review mode, append a `21-Hour Problem-First Review Roadmap` section after the main topic content and past exam analysis. Keep `Quick Reference` as the final compact lookup section.
- In roadmap-only mode, scan the folder and build the concept-to-question map, but read lecture material only as needed to connect questions back to source pages and concepts.
- In current-scope past exam map mode, scan all past exam papers down to subquestion level, judge each subquestion against the current course scope, and keep all subquestions in the output with a scope status.
- In outline-refinement mode, do not regenerate the whole review note unless the user asks. Audit the existing outline against the course materials, propose changes, interact with the user on priorities, then write the refined outline.
- If the user requests multiple modes, combine them in the order requested.

## Folder Scan

Before reading content deeply, list the target course folder recursively up to 2 levels deep and classify available PDFs plus any existing outline/review Markdown or text files. Use subfolder names, filenames, and visible document clues.

| Category | Common patterns | Use |
|---|---|---|
| Syllabus / Outline | `syllabus`, `outline`, `course_outline`, `module_guide` | Topic order, assessment structure, exam format |
| Existing review outline | `review_outline`, `revision_outline`, `复习大纲`, `大纲`, `roadmap`, `review.md`, `outline.md` | Candidate for outline-refinement mode |
| Lecture slides | `lecture`, `lecture_notes`, `slides`, `课件`, `lec`, `week`, `topic` | Primary source, required for folder-level runs |
| Problem sets | `problem_set`, `pset`, `ps`, `homework`, `hw`, `exercises` | Tested concepts and question patterns |
| Seminar / Tutorial | `seminar`, `tutorial`, `习题课`, `section` | Extra intuition, worked examples, common mistakes |
| Solutions | `solution`, `solutions`, `sol`, `answers`, `mark_scheme` | Expected solution structure and techniques |
| Past exams | `past_exam`, `past_paper`, `exam`, `final`, `midterm`, `mock` | Exam format and recurring topic patterns |
| Reading lists | `reading`, `references`, `bibliography` | Context only, do not summarise readings by default |

Handle folder layouts flexibly:

- Organized folders: classify first by folder name, then refine by filename.
- Flat folders: classify by filename patterns.
- Ambiguous files: list as `unclassified`; do not guess.
- Missing lectures in a folder-level task: stop and ask which files contain the lecture content.

Report the scan before proceeding, adapting to what exists:

```text
Found in EC487/:
  Syllabus: syllabus.pdf
  Lectures: lecture_notes/ (8 PDFs)
  Problem Sets: pset/ (4 PDFs)
  Solutions: solutions/ (4 PDFs)
  Past Exams: past_exam/ (3 PDFs)
  Seminar: not found
  Unclassified: notes_misc.pdf
```

## Resource Integration

Use available materials as one integrated evidence base, not as separate summaries.

| Available materials | Expected output |
|---|---|
| Lectures only | Concepts, formulas, models, proofs, diagrams, and intuition |
| Lectures + syllabus | Above, ordered by syllabus topics and assessment structure |
| Lectures + problem sets | Above, plus "how it is tested" per topic |
| Lectures + solutions | Above, plus answer structure, rigor level, and techniques |
| Lectures + seminars | Above, plus extra intuition and common mistakes |
| Lectures + past exams | Above, plus exam format, recurring topics, and question-type patterns |

When extra resources exist beside the lectures, integrate them into the relevant topic blocks. Producing a lecture-only summary while ignoring available problem sets, solutions, seminars, or exams is a failure.

## Workflow

1. **Scan and classify.** Report found materials and note any missing or ambiguous categories.
2. **Read syllabus first if present.** Extract topic sequence, lecture-to-topic mapping, assessment format, examinable topics, and readings. Use it as the structural backbone.
3. **Build a lecture map.** Read enough of all lectures to identify lecture boundaries, topic transitions, summary slides, and page ranges. For long decks, segment before deep extraction.
4. **Build a concept-to-question map if practice materials exist.** Skim problem sets, past exams, seminars, and solutions. Record question ID, topic, concept tested, question type, and any solution technique.
5. **Deep-read by topic segment.** Extract definitions, formulas, model setups, propositions, proof strategies, diagrams, examples, techniques, and common mistakes. Attach page references immediately while reading.
6. **Merge into one review document.** Organize by syllabus topic order if available, otherwise by inferred lecture topic order. Put practice patterns and answer techniques under the concepts they test.
7. **Add final sections.** Include `Past Exam Analysis` if past exams exist, add the problem-first roadmap unless the user opted out, and always include a compact `Quick Reference` section.
8. **Run the verifier pass.** Fix gaps before writing the final file.
9. **Write the Markdown file.** Save `<stem>_review.md` directly in the course directory and confirm it exists.

For roadmap-only mode, run Steps 1-4, then jump to `Problem-First Roadmap`. For current-scope past exam map mode, run Steps 1-4, enumerate all past exam questions and subquestions, then jump to `Current-Scope Past Exam Map`. For outline-refinement mode, run Steps 1-4 as an audit pass, then jump to `Existing Outline Refinement`.

## Output Contract

### File Naming

Use one output file in the course directory, with this priority:

1. Course code from folder or syllabus: `EC487_review.md`
2. Folder name if course code is unclear: `Advanced_Micro_review.md`
3. Source PDF stem for single-PDF runs: `labor_econ_slides_review.md`
4. Fallback: `econ_review.md`

Use a custom filename if the user specifies one.

Roadmap-only output uses `<stem>_roadmap.md`. Current-scope past exam map output uses `<stem>_current_scope_exam_map.md`. Outline refinement output uses `<outline-stem>_refined.md` by default and must not overwrite the original outline unless the user explicitly asks for that.

### YAML Frontmatter

```yaml
---
title: "<course-code> Review"
date: <YYYY-MM-DD>
course: "<course code and name>"
tags:
  - review
  - <course-tag>
---
```

Include `syllabus_topics` and `exam_format` only when that information was found.

### Document Structure

```markdown
# <Course Code> 复习笔记

## Exam Information
## Topic 1: <name>
### Key Concepts
### Models
### Formulas
### Problem Patterns
### 易错点与解题技巧
## Topic 2: <name>
## Past Exam Analysis
## Current-Scope Past Exam Summary
## 21-Hour Problem-First Review Roadmap
## Quick Reference
```

Skip unavailable sections rather than filling them with boilerplate. For example, omit `Past Exam Analysis` when no past exams were found.

### Heading-Based Blocks

Do not use Obsidian-style callout or admonition markers. Use plain Markdown headings so the output works cleanly in Obsidian, GitHub, PDFs, Word exports, and printed notes.

Use these headings consistently:

- `#### Definition`: formal definitions, preserving exact mathematical content when possible.
- `#### Formula`: key formulas with conditions for validity.
- `#### Economic Intuition`: mandatory economic intuition after formal results.
- `#### Problem Pattern`: representative problem patterns from problem sets, seminars, or exams.
- `#### Answer Technique`: answer structures, derivation tricks, and problem-solving methods.
- `#### Common Mistakes`: fragile assumptions, tricky edge cases, and frequent errors.
- `#### Exam Evidence`: past exam observations and likely tested patterns.
- `#### Diagram Notes`: textual descriptions of important diagrams.

For compact topics, combine related blocks under one concept heading, but never omit `#### Economic Intuition` after a formal result.

If formatting examples are needed, read `references/heading-output-examples.md`.

### Citations and Practice Links

- Put citations at the point level, not just section level.
- Wrap source locators in Markdown inline-code backticks. Use:

```markdown
`课件页 16`
`课件页 12-18`
`Lec 3 课件页 12-18`
`PS2 Q3`
`Exam 2023 Q1b`
```

- Append recommendations inside the same source tag only when matched evidence exists:

```markdown
`课件页 16｜推荐练习：PS2 Q3, Exam 2023 Q1b`
```

- If a formal result appears in a non-lecture source only, cite that source precisely and avoid pretending there is a slide page.

### Current-Scope Past Exam Map

Use this mode when the user asks whether past exam questions are still examinable under the current course scope, or asks to map past exam questions back to current lecture slides.

Current scope basis:

1. Prefer the current syllabus, module guide, course outline, or explicit current review outline when available.
2. If no syllabus/outline exists, treat the current lecture PDFs as the source of truth.
3. Use current problem sets as supporting evidence for practice alignment, not as the sole scope authority unless lectures are ambiguous.

Past exam handling:

- Traverse every past exam paper found in `past_exam`, `past_paper`, `exam`, `final`, `midterm`, or equivalent folders.
- Enumerate questions down to subquestion level, such as `Exam 2023 Q1b`.
- Keep every past exam subquestion in the output. Do not drop out-of-scope questions.
- Do not solve the exam question. Extract the tested concept, required technique, current relevance, lecture location, and related practice.

Scope status labels:

- `In Scope`: the concept appears in the current syllabus or current lecture slides.
- `Partially In Scope`: part of the subquestion matches current content, but another part uses removed or extra material.
- `Out of Scope`: the tested concept does not appear in current syllabus or current lecture slides.
- `Unclear`: the exam subquestion or current lecture mapping is too ambiguous to classify safely.

Lecture location rules:

- Prefer exact lecture page locators, such as `Lec 6 课件页 34-42`.
- If exact pages are uncertain, use the closest lecture section/topic, such as `Lec 6 Section: Incentive Compatibility`.
- Do not invent page numbers. Section-level fallback is better than false precision.

Standalone output structure:

```markdown
# Current-Scope Past Exam Map

## Current Scope Basis

- Syllabus / outline used: ...
- Current lecture slides used: ...
- Scope inference rule: ...

## Exam-to-Lecture Coverage Table

| Exam | Subquestion | Topic Tested | Scope Status | Current Lecture Location | Related Problem Set | Notes |
|---|---|---|---|---|---|---|

## Question-by-Question Mapping

### Exam 2023 Q1b

#### Scope Status

In Scope / Partially In Scope / Out of Scope / Unclear

#### Tested Knowledge Point

...

#### Current Lecture Location

`Lec 6 课件页 34-42`

#### Related Problem Set

`PS2 Q3`

#### Mapping Note

Explain briefly why this subquestion is in scope or out of scope.
```

In full review mode, include only a compact `Current-Scope Past Exam Summary` with high-value in-scope patterns and major out-of-scope warnings. Keep the full per-subquestion table for standalone current-scope map mode.

### Problem-First Roadmap

The roadmap is a fixed-budget revision plan for 21 hours by default: 7 sessions, 3 hours each. If the user asks for another time budget, scale the sessions while preserving the problem-first logic.

Build the plan from practice evidence in this priority order:

1. Past exams: highest priority for format, topic recurrence, and exam-style timing.
2. Problem sets: highest priority for tested techniques and topic coverage.
3. Solutions and seminars: use for answer structure, common shortcuts, and mistakes.
4. Lectures: use to backfill weak concepts and cite source pages after a question reveals the gap.

Each session should include:

- `Goal`: what exam ability this session builds.
- `Start from questions`: specific past exam or problem set questions to attempt first.
- `Return to notes`: exact lecture/source pages to revisit after the attempt, using inline-code source locators.
- `Fix the gap`: formulas, models, proof techniques, or intuition to repair.
- `Output`: a concrete artifact such as an error log, formula sheet update, rewritten answer plan, or timed redo.

Session structure for each 3-hour block:

| Time | Task |
|---|---|
| 0:00-0:20 | Preview the target question set and write what each question is testing |
| 0:20-1:35 | Attempt questions under light timing without notes |
| 1:35-2:20 | Check solutions or seminar notes and identify gaps |
| 2:20-2:50 | Return to lecture pages and patch concepts/formulas |
| 2:50-3:00 | Record mistakes and choose the next redo target |

If no past exams or problem sets exist, say that the plan is lecture-driven due to missing practice evidence, then create retrieval tasks from lecture examples instead of inventing fake questions.

### Existing Outline Refinement

Use this mode when the user asks to modify or polish an existing review outline, or points to an outline file.

1. Identify candidate outline files from the scan. If multiple plausible outlines exist, ask which one to edit.
2. Read the chosen outline, syllabus if available, and the concept-to-question map.
3. Audit the outline for missing high-yield topics, weak intuition, missing formulas, missing page locators, missing practice links, poor sequencing, and overlong prose.
4. Present a concise audit to the user with suggested edits and ask for their preferred direction if not already specified: exam-oriented, intuition-first, concise cheat sheet, or comprehensive review.
5. Apply the agreed edits. Preserve the user's useful structure and wording where possible.
6. Save a refined copy by default. Do not overwrite the original outline unless explicitly requested.

The refined outline should make the student's problem-first workflow visible: each topic should show representative questions first, then the lecture concepts and pages needed to answer them.

### Math and Style

- Inline math: `$...$`; display math: `$$...$$`.
- Use standard LaTeX supported by Obsidian MathJax, including `aligned` and `cases`.
- Prefer short bullets and compact heading blocks. Avoid large prose blocks except where intuition genuinely needs 1-3 sentences.
- Do not use Obsidian wikilinks, HTML tags, embedded images, or nested bullets deeper than 3 levels.
- Use Markdown tables for payoff matrices and compact comparisons.

## Economic Intuition

Treat intuition as the central product, not as decoration. For each result, explain at least one of:

- Why the result holds economically: incentives, constraints, surplus, trade-offs, efficiency, information, selection, or identification.
- What assumption drives the result and what would change if it failed.
- What real-world setting the model clarifies.
- Why the mathematical step has economic content.

Good intuition explains the mechanism. Bad intuition merely says `由公式可得`, restates the math in Chinese, or says the point matters because it may be on the exam.

## Content Extraction Rules

### Models

For each economic model, extract:

1. Setup: agents, choice variables, constraints, timing, information, objectives.
2. Solution concept: Nash, Bayesian, Walrasian, competitive equilibrium, planner problem, IC/IR, or other relevant concept.
3. Key result: proposition, comparative static, welfare implication, or identification result.
4. Intuition: why the result holds.
5. Tested pattern: matching problem set, seminar, or exam question if available.

### Proofs and Derivations

- Preserve the proof strategy: contradiction, FOC/SOC, envelope theorem, fixed point, law of iterated expectations, IV moment condition, and so on.
- Keep exam-relevant intermediate steps and named tricks.
- Summarize long derivations as starting point -> key transformation -> result.
- Explain why the strategy works economically, not only algebraically.

### Econometrics

- Distinguish population parameter, estimator, and estimate.
- State estimator, assumptions, identifying variation, properties, and failure modes.
- For hypothesis tests, record null, statistic, distribution under the null, decision rule, and interpretation.
- Preserve exact variance, standard error, clustering, robust SE, and asymptotic formulas when shown.
- Explain what source of variation identifies the coefficient and why assumptions may fail.

### Game Theory and Mechanism Design

- State players, strategy spaces, payoffs, timing, information, and equilibrium concept.
- Write equilibrium strategy profiles explicitly.
- For mechanism design, record social choice function, mechanism, IC, IR, efficiency, allocation rule, and payment rule.
- Explain strategic incentives and why the equilibrium or mechanism survives deviation incentives.

### Diagrams

Use `#### Diagram Notes` blocks for diagrams that matter. Describe axes, curves, equilibrium points, shifts, comparative statics, and the economic meaning of movement in the figure.

## Verifier Pass

Before finalizing, check:

- Folder scan was reported, or single-PDF mode was explicitly used.
- Syllabus topics or inferred lecture topics are all covered.
- No lecture block was silently dropped.
- Available problem sets, solutions, seminars, and past exams were integrated into relevant topics.
- The requested mode was respected; roadmap-only and outline-refinement requests did not accidentally produce a full review note.
- Current-scope past exam map mode produced only `<stem>_current_scope_exam_map.md`, unless the user requested combined modes.
- Current scope basis is explicit: syllabus/outline if available, otherwise current lecture PDFs.
- Every past exam subquestion is retained with `In Scope`, `Partially In Scope`, `Out of Scope`, or `Unclear`.
- Ambiguous mappings use section-level fallback instead of invented page numbers.
- Full review mode includes a 21-hour problem-first roadmap unless the user opted out.
- Roadmap tasks start from past exams/problem sets where available, then return to lecture pages.
- Outline refinement preserves the original outline by default and saves a refined copy.
- Every formal result has a `#### Economic Intuition` subsection or an explicit `直觉待补充` note.
- Every knowledge point has a precise source locator.
- Formula-heavy sections include formulas, assumptions, and interpretation.
- Problem recommendations are present wherever the concept-to-question map found a match.
- English terminology appears beside important Chinese terms.
- Math is Markdown/MathJax-renderable.
- The output file exists on disk.

## Failure Modes

- Jumping into one PDF before scanning a course folder.
- Ignoring a syllabus that defines topic order or exam format.
- Summarising resources separately instead of integrating them by topic.
- Losing page references during chunking.
- Adding fake practice links.
- Generating a lecture-first roadmap when past exams or problem sets are available.
- Dropping out-of-scope past exam subquestions instead of retaining and labeling them.
- Inventing exact lecture page numbers when only the section can be identified.
- Treating old past exam coverage as current scope without checking current syllabus or lecture slides.
- Overwriting an existing outline without permission.
- Ignoring the user's mode request and producing a full note when they asked only for a roadmap or outline refinement.
- Writing generic prose instead of exam-useful concepts, formulas, techniques, and mistakes.
- Treating `由公式可得` as economic intuition.
- Stopping before writing the `.md` artifact.

---
> Source: [AugustHuangMX/econ-review-master](https://github.com/AugustHuangMX/econ-review-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
