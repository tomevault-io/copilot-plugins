## course-cheatsheet-maker

> Use this skill when the user wants Codex to analyze course knowledge sources and question sources organized as materials/knowledge and materials/questions, infer assessment source types, estimate exam topic importance, and generate compact A4 landscape printable cheatsheets in HTML, with optional manual PDF export from the browser. Trigger for exam review, midterm/final review, one-page cheatsheet, formula sheet, bilingual Chinese-English review sheet, topic importance ranking, Safe Review Mode, or Full Auto Mode. Do not use for unrelated summarization.


# course-cheatsheet-maker

Use this skill to help Codex analyze course materials, rank topic importance from evidence, and produce compact A4 landscape cheatsheets. Codex/the agent does the reasoning; scripts only extract source text, render already prepared content, or validate workflow contracts.

## When To Use

Use for exam review, midterm/final review, one-page cheatsheets, formula sheets, bilingual Chinese-English review sheets, topic maps, and topic importance ranking.

Do not use for unrelated summarization, generic study notes, or claims about what will definitely appear on an exam.

## Inputs

Use only this simplified material structure for real courses:

```text
courses/COURSE_NAME/
  materials/
    knowledge/
    questions/
  working/
  outputs/
```

Do not ask the user to sort materials into many detailed folders. The user only needs `knowledge/` and `questions/`. Subfolders inside those two folders are allowed.

## Parameter Model

Use five separate parameters:

- `workflow_mode`: `safe-review` or `full-auto`. This controls automation only. It is selected by the user request or CLI `--mode`; do not treat Full Auto as a global content default.
- `layout`: `3col` or `4col`. This selects the preferred rendered layout for validation and review. Full Auto still renders both HTML files.
- `target_pages`: integer bounded content/page budget. User-facing "4+" means `target_pages >= 4`; scripts and `run_config.md` must record an integer.
- `coverage_mode`: `exam-compact`, `balanced-standard`, or `comprehensive-review`. This controls which topics are included.
- `detail_mode`: `simple`, `balanced`, or `detailed`. This controls how much detail included topics receive.

Resolve missing parameters before extraction and write `working/run_config.md` first so the run goal is established before material processing.

Defaults:

```text
workflow_mode = safe-review
layout = 3col
target_pages = 1
coverage_mode = balanced-standard
detail_mode = balanced
```

Special comprehensive default: if the user asks for "comprehensive", "full review", "cover all slides", "全面复习", or similar without specifying page count, use `coverage_mode: comprehensive-review` and `target_pages: 2`. Keep `detail_mode: balanced` unless the user explicitly asks for detailed output.

Natural language inference examples:

- "one-page", "一页" -> `target_pages = 1`
- "two-page", "两页" -> `target_pages = 2`
- "three-page", "三页" -> `target_pages = 3`
- "final exam", "midterm", "考试速记", "只要重点" -> `coverage_mode = exam-compact`
- "standard review", "正常复习版", "标准版" -> `coverage_mode = balanced-standard`
- "comprehensive", "全面复习", "cover all slides" -> `coverage_mode = comprehensive-review`
- "simple", "concise", "quick reference", "简洁", "速查" -> `detail_mode = simple`
- "detailed", "explain more", "学习版", "详细版" -> `detail_mode = detailed`

Resolve conflicts in this priority order:

1. `target_pages`
2. `layout`
3. `coverage_mode`
4. `detail_mode`
5. `workflow_mode`

`target_pages` controls total content capacity, not an exact printed-page guarantee and not the per-topic writing depth. For example, if the user asks for a "one-page detailed comprehensive review", keep `target_pages = 1` and preserve requested detail for A topics first while compressing lower-priority content.

Coverage modes:

- `exam-compact`: A topics must be included; B topics should fit if space allows; C topics only if space allows; R topics omitted. Past-paper evidence has the strongest influence. Lecture-only topics may be compressed or omitted unless foundational.
- `balanced-standard`: default content coverage. A topics must be included; B topics should be included; C topics should be representatively included; R topics omitted. Lecture coverage decides minimum inclusion, while exam evidence strongly increases priority and usually justifies expanded treatment.
- `comprehensive-review`: not constrained to one A4 page. A and B topics must be included; C topics should be included if useful; R topics are generally omitted unless explicitly useful as appendix/reference material.

Representative C-topic rule: for `coverage_mode = comprehensive-review` and `target_pages >= 2`, include representative C topics when meaningful C candidates exist. Do not reduce C topics to zero unless there are no meaningful C candidates. Compress repeated examples, repeated traps, and Q/A before removing all representative C topics. For a 2-page `3col` comprehensive review, aim for 6-12 representative C topics where useful.

Target page budget policy for 3col:

- `target_pages = 1`: floor 14k chars, target 16k-18k, soft ceiling 20k, hard ceiling 22k; topic target 35-45; A all compact, B selected high-value, C 0-3 representative, R omit.
- `target_pages = 2`: floor 34k chars, target 36k-38k, soft ceiling 40k, hard ceiling 41k; topic target 75-85; A all, B most important, C 6-12 representative, R omit.
- `target_pages = 3`: floor 44k chars, target 48k-52k, soft ceiling 55k, hard ceiling 58k; topic target 95-115; A all, B nearly all, C 18-30 representative/useful, R 0-3 useful reference items only.
- `target_pages = 4`: floor 60k chars, target 64k-70k, soft ceiling 74k, hard ceiling 78k; topic target 115-135; A all, B all or nearly all, C most useful C topics, R 3-8 useful reference items only.
- `target_pages = 5`: floor 76k chars, target 82k-88k, soft ceiling 93k, hard ceiling 98k; topic target 130-150; A all, B all, C most or nearly all meaningful C topics, R 5-12 useful reference items only.
- `target_pages = 6`: floor 92k chars, target 100k-108k, soft ceiling 114k, hard ceiling 120k; topic target 145-170; A all, B all, C nearly all meaningful C topics, R useful appendix/reference only.

These are guidance ranges, not hard content-generation requirements. Do not invent topics to hit a topic count. If available candidate units are fewer than the topic target, use remaining space for compact examples, traps, comparisons, and Q/A.

Detail mode maps priority to verbosity:

| Priority | detailed | balanced | simple |
| --- | --- | --- | --- |
| A | detailed | detailed | detailed |
| B | detailed | detailed | simple |
| C | detailed | simple | minimal/simple |
| R | omit | omit | omit |

Priority equals importance only. Detail mode equals verbosity only. Do not downgrade A to B because a topic can be summarized briefly.

Correct:

```text
Topic: RGB and YUV color spaces
Score: 12
Priority: A
Detail under simple mode: simple
Reason: high-importance lecture concept, but can be summarized compactly
```

Incorrect:

```text
Topic: RGB and YUV color spaces
Score: 12
Priority: B
Reason: only needs simple treatment
```

Lecture coverage decides minimum inclusion. Exam evidence strongly increases priority and usually justifies expanded treatment. Final verbosity is controlled by `detail_mode`.

Each major knowledge PDF should normally have at least 2-4 represented concepts unless it is administrative, repetitive, or low-testability. This coverage floor is a minimum safeguard, not a maximum cap: it must never cap A/B topics or high-importance lecture concepts.

`target_pages` must not directly change `detail_mode`. For a fixed `detail_mode`, increasing `target_pages` should first increase topic coverage. After coverage is sufficient, extra space can be used for high-value support blocks: traps, compact examples, comparison tables, and Q/A. For example, `target_pages=6` and `detail_mode=balanced` should still use balanced topic writing, not automatically turn every topic into detailed prose.

When `target_pages` is fixed, `detail_mode` controls per-topic expansion depth:

- `simple`: shorter explanations, so more topics can fit.
- `balanced`: balance topic coverage and explanation depth.
- `detailed`: A/B topics get fuller explanations, examples, traps, and comparisons; fewer C topics may fit.

Do not add filler text just to fill pages. If the output under-fills `target_pages`, expand in this order: missing A/B topics, representative C topics, common traps and confusions, compact examples, comparison tables, then Q/A blocks. If the page budget is tight, preserve requested detail for A topics first, then compress B/C topics and optional examples.

Page-budget validation is separate from priority ranking. For `3col`, use the calibrated 1-6 page character ranges above. For `target_pages > 6`, validation falls back to conservative extrapolation and warns that the range is less calibrated. For `4col`, keep the older column-slot heuristic as a less-tested fallback. Optional print-page validation is final verification only; do not repeatedly loop through expensive print checks unless explicitly requested.

Do not over-merge candidate units into very broad topics. This is especially important when `target_pages >= 2`, `coverage_mode = comprehensive-review`, or `detail_mode = detailed`. For multi-page or comprehensive modes, preserve subtopics as selectable units when they are useful separately, such as sensor categories, sensor limitations, touch sensing, microphones, sampling, quantization, reproduction devices, and human perception limits.

## Knowledge Sources

Put lecture/content sources in `materials/knowledge/`, such as lecture slide decks, PPTs, lecture notes, review notes, reading notes, textbook summaries, and instructor review guides.

Use these sources to identify course emphasis, repeated explanations, definitions, formulas, algorithms, procedures, comparison pairs, and common traps. Instructor review guides, final review slides, official revision checklists, and exam revision notes are high course-emphasis evidence, but still must not be described as guaranteed exam content.

## Question Sources

Put question-like or assessment-related sources in `materials/questions/`, such as past papers, final exams, sample exams, mock exams, quizzes, assignments, tutorials, workshops, exercise sheets, and problem sets.

Codex must infer the specific source type from filenames and extracted content. If a file has an unclear type, mark it uncertain and do not over-weight it.

## Style Reference

If present, inspect:

```text
courses/_style_reference/sample_cheatsheet.docx
```

Use it only to infer layout and writing style: A4 landscape, narrow margins, dense Chinese-English notes, 3-column or 4-column layout, small font, compact line height, formulas, short Q/A blocks, traps, and comparison points.

Never copy course content from the style reference into a generated cheatsheet.

## Required Working Files

Create these files during the agent reasoning phase before rendering:

```text
courses/COURSE_NAME/working/run_config.md
courses/COURSE_NAME/working/candidate_units.jsonl
courses/COURSE_NAME/working/knowledge_units.md
courses/COURSE_NAME/working/topic_map.md
courses/COURSE_NAME/working/importance_ranking.md
courses/COURSE_NAME/working/topic_evidence_map.md
courses/COURSE_NAME/working/cheatsheet_content.md
```

`run_config.md` is created before extraction. It records resolved run parameters, user-specified parameters, defaulted parameters, conflict resolution notes, and the selection plan.

`candidate_units.jsonl` is generated by `build_candidate_units.py` from extracted Markdown. It is a deterministic, coarse chunking and traceability layer, not semantic ranking.

`knowledge_units.md` is generated by Codex/the agent from `candidate_units.jsonl`. It refines, merges, classifies, and scores candidate units before topic mapping and importance ranking.

`topic_evidence_map.md` is a concise review and traceability artifact. It is not part of the printable cheatsheet unless the user explicitly asks.

`build_html.py` reads only `cheatsheet_content.md`. It does not analyze course materials, read extracted text, read `topic_evidence_map.md`, create topic rankings, or decide detail modes.

Suggested `run_config.md` structure:

```text
# Run Configuration

workflow_mode: full-auto
layout: 3col
target_pages: 2
coverage_mode: comprehensive-review
detail_mode: balanced

## User-specified parameters
- User requested comprehensive review.

## Defaulted parameters
- layout defaulted to 3col.
- target_pages defaulted to 2 because coverage_mode is comprehensive-review.
- detail_mode defaulted to balanced.

## Conflict resolution
- None.

## Selection Plan
Target pages: 2

Expected inclusion:
- A topics: include all
- B topics: include most
- C topics: include representative topics
- R topics: omit unless needed as reference

Expected expansion:
- A topics: definition + formula/procedure + trap/example
- B topics: compact explanation
- C topics: one-line summary or table row
```

## Skill Code Editing Rule

During normal course cheatsheet generation, do not modify scripts in the installed skill scripts directory unless the user explicitly asks to modify the reusable skill itself. Course-specific analysis should update only the course `working/` and `outputs/` artifacts.

## Material Extraction Phase

Before topic analysis, run:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/extract_materials.py" COURSE_NAME --root courses
```

Before extracting files, `extract_materials.py` checks the current Python interpreter for optional binary extraction dependencies listed in `requirements.txt`:

```text
pymupdf
python-pptx
python-docx
```

The checker uses `sys.executable`, verifies imports `fitz`, `pptx`, and `docx`, and installs missing packages with:

```bash
python -m pip install -r "$SKILL_DIR/requirements.txt"
```

Set `COURSE_CHEATSHEET_NO_AUTO_INSTALL=1` to skip automatic installation and keep graceful missing-dependency behavior. If installation fails, extraction still writes `working/extraction_report.md` with clear per-file failure messages. OCR dependencies are not installed; OCR remains out of scope.

The extractor recursively scans:

```text
courses/COURSE_NAME/materials/knowledge/
courses/COURSE_NAME/materials/questions/
```

It writes:

```text
courses/COURSE_NAME/working/extracted/knowledge/
courses/COURSE_NAME/working/extracted/questions/
courses/COURSE_NAME/working/extraction_report.md
```

Extracted Markdown should preserve default source locators:

- PDF: `## Page N` for every page in PyMuPDF page order. If a page has no selectable text, include `<!-- No selectable text on this page. -->`. Do not add OCR.
- PPTX: `## Slide N` for every slide, with visible text grouped by slide and tables included where practical. Do not infer importance from slide titles.
- DOCX: keep useful structure such as `## Document Text`, `### Heading: ...`, `Paragraph: ...`, and `Table:`.
- TXT/MD: preserve original text as much as possible; add lightweight `## Chunk N` markers only for long files when useful.

`extraction_report.md` should include stable source IDs within the run:

- Knowledge sources: `K01`, `K02`, ...
- Question sources: `Q01`, `Q02`, ...
- Include file, folder, inferred type, extracted unit count, status, output path, and message.

Codex should analyze `working/extracted/` and `working/extraction_report.md` whenever available. Do not infer detailed document content from filenames alone. If extraction fails, if a PDF has no selectable text, or if only filenames are available, label topic ranking, coverage, and cheatsheet content as preliminary.

Scanned PDFs may have no selectable text. OCR is out of scope.

## Candidate And Knowledge Units

After extraction and before topic analysis, run:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/build_candidate_units.py" COURSE_NAME --root courses
```

The script reads extracted Markdown and writes:

```text
courses/COURSE_NAME/working/candidate_units.jsonl
```

Every candidate unit must clearly identify whether it came from knowledge material or question/evidence material:

- `source_folder`: `knowledge` or `questions`
- `inferred_source_type`: for example `knowledge_source`, `final_exam`, `quiz`, or `unknown_question_source`
- `unit_role`: `knowledge_unit` or `question_evidence_unit`

Question units are evidence units. Do not treat them as normal lecture concepts. Use them to support `Exam evidence` and source-type inference during ranking.

Codex then creates:

```text
courses/COURSE_NAME/working/knowledge_units.md
```

`knowledge_units.md` should refine and merge candidate units into semantic knowledge units. Each unit should include:

- Unit ID
- Concept
- Source and page/slide/chunk range
- Unit type
- Topic type
- Lecture coverage
- Exam evidence
- Testability
- Dependency
- Error risk
- Suggested priority
- Suggested detail
- Inclusion reason

Suggested topic types:

- `core_exam`: supported by question evidence and lectures; usually formulas, procedures, traps, and worked examples.
- `lecture_coverage`: heavily covered in lectures but little/no exam evidence; usually definitions, comparisons, mechanisms, or applications.
- `backup_example`: application cases, examples, or extended context; usually compact example-bank treatment.
- `foundational`: needed to understand later content; keep even if not directly examined.
- `administrative`: course logistics, grading, syllabus details; normally omitted from the cheatsheet.

## Required HTML Outputs

Full Auto Mode renders:

```text
courses/COURSE_NAME/outputs/cheatsheet_3col.html
courses/COURSE_NAME/outputs/cheatsheet_4col.html
```

HTML/CSS is the source of truth. PDF export is optional and may be done manually from the browser.

## Safe Review Mode

Use when the user wants more control.

1. Resolve `workflow_mode`, `layout`, `target_pages`, `coverage_mode`, and `detail_mode`.
2. Create `working/run_config.md`.
3. Run `$SKILL_DIR/scripts/extract_materials.py COURSE_NAME --root courses`.
4. Run `$SKILL_DIR/scripts/build_candidate_units.py COURSE_NAME --root courses` unless the user intentionally pauses immediately after extraction.
5. Analyze `working/run_config.md`, `working/candidate_units.jsonl`, `working/extracted/`, and `working/extraction_report.md`.
6. Infer source types inside `questions/` from filenames and extracted content.
7. Create `working/knowledge_units.md`, `working/topic_map.md`, `working/importance_ranking.md`, `working/topic_evidence_map.md`, and `working/cheatsheet_content.md`.
8. Stop for user review.
9. Do not run `build_html.py`.

If the user explicitly asks to skip traceability artifacts in Safe Review Mode, `topic_evidence_map.md` may be deferred. Full Auto Mode should create it.

## Full Auto Mode

Use when the user wants fast output.

1. Resolve `workflow_mode`, `layout`, `target_pages`, `coverage_mode`, and `detail_mode`.
2. Create `working/run_config.md`.
3. Run `$SKILL_DIR/scripts/extract_materials.py COURSE_NAME --root courses`.
4. Run `$SKILL_DIR/scripts/build_candidate_units.py COURSE_NAME --root courses`.
5. Analyze `working/run_config.md`, `working/candidate_units.jsonl`, `working/extracted/`, and `working/extraction_report.md`.
6. Infer source types inside `questions/` from filenames and extracted content.
7. Create `knowledge_units.md`, `topic_map.md`, `importance_ranking.md`, `topic_evidence_map.md`, and `cheatsheet_content.md` first as traceable reasoning artifacts.
8. Run `$SKILL_DIR/scripts/build_html.py COURSE_NAME --root courses`.
9. Generate both HTML outputs.
10. Run `$SKILL_DIR/scripts/validate_workflow.py COURSE_NAME --root courses --mode full-auto --layout LAYOUT --target-pages TARGET_PAGES --coverage-mode COVERAGE_MODE --detail-mode DETAIL_MODE`.
11. If validation fails, fix the cause and rerun from the appropriate earlier step.
12. If validation reports actual printed-page underfill, character-count underfill, sparse target output, or over-compression warnings, revise `cheatsheet_content.md` once before finalizing unless the warning is clearly a false positive. Updating only `importance_ranking.md`, `topic_map.md`, `run_config.md`, or `topic_evidence_map.md` is not a successful repair. If a repair pass is triggered but `cheatsheet_content.md` does not change, report repair failure instead of claiming success; after changing `cheatsheet_content.md`, rerender the HTML outputs.
13. If validation reports mojibake in bilingual Topic headings, repair `cheatsheet_content.md` with valid UTF-8 Chinese labels and rerender the HTML outputs. Do not treat mojibake headings as valid Chinese output.
14. Summarize uncertainty, incomplete materials, weak-evidence topics, extraction status, resolved parameters, and validation results.

`validate_workflow.py` validates reliability and output contracts. It does not replace Codex analysis.

## Source-Type Inference For Questions

- `past_paper_2023.pdf`, `final_exam_2022.pdf`, `exam_2021.pdf`: past/final exam; strongest question evidence.
- `sample_exam_2024.pdf`, `mock_exam.pdf`, `practice_exam.pdf`: sample/mock/practice exam; strongest or high question evidence.
- `quiz_week3.pdf`, `quiz_*.pdf`: quiz; strong testability evidence.
- `assignment_2.pdf`, `homework_*.pdf`: assignment; medium-to-strong practice evidence.
- `problem_set_4.pdf`, `exercise_sheet_*.pdf`: problem set/exercise sheet; medium-to-strong practice evidence.
- `workshop_5_questions.pdf`: workshop; practice evidence.
- `tutorial_week4.pdf`: tutorial; practice evidence.
- `unknown_questions.pdf` or unclear names: unknown question source; mark uncertain and do not over-weight unless extracted content clearly resembles an exam, quiz, assignment, tutorial, workshop, or problem set.

Extractor metadata uses normalized question source types: `past_paper`, `final_exam`, `sample_exam`, `mock_exam`, `practice_exam`, `quiz`, `assignment`, `homework`, `problem_set`, `exercise_sheet`, `workshop`, `tutorial`, and `unknown_question_source`.

## Topic Map Instructions

In `topic_map.md`, include:

- Source inventory grouped by `knowledge/` and inferred `questions/` source type.
- Main topics and subtopics.
- Where each topic appears, using source IDs and page/slide/chunk markers when available.
- Related formulas, definitions, procedures, traps, examples, and likely problem forms.
- Notes about uncertain source types, missing sources, or incomplete materials.

## Importance Ranking Instructions

In `importance_ranking.md`, use `references/importance_rubric.md`.

Each topic must include:

```text
Priority: A/B/C/R
Topic: English topic / 中文标签 [A]
Topic type: core_exam/lecture_coverage/backup_example/foundational/administrative
Scores: Lecture coverage=?, Exam evidence=?, Testability/formula value=?, Dependency=?, Error risk=?, Total=?
Evidence: brief source summary
Lecture coverage: low/medium/high with source IDs
Exam evidence: none/weak/medium/high with question source IDs
Recommended detail: detailed/simple/minimal/omit
Rationale: short reason
```

Score dimensions are 0-3 each: `Lecture coverage`, `Exam evidence`, `Testability/formula value`, `Dependency`, and `Error risk`. Total max score is 15. `Testability/formula value` includes reusable formulas, procedures, algorithms, comparisons, or calculation value.

Priority bands:

- A: 11-15.
- B: 7-10.
- C: 4-6.
- R: 0-3.

Priority must match the score band unless an explicit allowed override is documented. Detail mode must not downgrade priority.

Override and cap rules:

- If `Exam evidence = 3` and `Testability/formula value >= 2`, the topic can be A even if total score is slightly below 11.
- If `Lecture coverage = 3`, `Testability/formula value >= 2`, and `Dependency >= 2`, the topic can be A even without past-paper evidence.
- Filename-only evidence cannot exceed C.
- Unknown question source cannot exceed B unless extracted content clearly proves exam, quiz, assignment, tutorial, workshop, or problem-set structure.
- If extraction failed or evidence is incomplete, ranking/content must be marked preliminary.
- No topic may be described as guaranteed exam content.

Past paper, final exam, sample exam, and mock exam evidence is the strongest question evidence. Instructor review guides, final review slides, official revision checklists, and exam revision notes are high course-emphasis evidence, but still not guaranteed exam content.

Do not put source coverage directly into the importance score unless there is a strong reason. Prefer source coverage as a post-ranking check: after initial ranking, check whether each major knowledge source is represented; if a major source is under-covered, promote the best candidate units from that source according to coverage mode and evidence quality.

If no past papers or sample exams are found in `questions/`, include this exact wording:

```text
没有提供往年卷，因此重要程度主要基于 PPT/Notes 强调、Quiz/Assignment/Workshop/Tutorial 出现频率、知识点基础性和可考性判断。
```

If materials are incomplete, clearly say the ranking is preliminary.

## Detail Mode Instructions

`detail_mode` is a writing/planning decision Codex makes while generating `importance_ranking.md` and `cheatsheet_content.md`. `build_html.py` must not infer or decide topic importance or detail level. Detail mode controls verbosity only; it must not decide whether a topic is important.

Supported values:

- `detail_mode: detailed`
- `detail_mode: balanced`
- `detail_mode: simple`

Detail mapping:

| Priority | detailed | balanced | simple |
| --- | --- | --- | --- |
| A | detailed | detailed | detailed |
| B | detailed | detailed | simple |
| C | detailed | simple | minimal/simple |
| R | omit | omit | omit |

If the user does not specify a detail preference, use `detail_mode: balanced`.

If the user asks for detailed output, use `detail_mode: detailed`.

If the user asks for comprehensive or full review without explicitly asking for detailed explanations, keep `detail_mode: balanced` and default `target_pages` to 2.

If the user asks for simple, short, one-page, or ultra-compact output, use `detail_mode: simple`.

Do not implement complex page fitting in `build_html.py`; make inclusion/detail choices in `knowledge_units.md`, `importance_ranking.md`, and `cheatsheet_content.md`.

## Topic Evidence Map Instructions

Create `working/topic_evidence_map.md` after `topic_map.md` and `importance_ranking.md`.

Purpose:

- Map major included topics to supporting knowledge/question sources.
- Help users verify where each topic came from.
- Keep printable cheatsheets compact by not embedding long citations in `cheatsheet_content.md`.

Scope:

- Include all A topics.
- Include B and C topics that appear in `cheatsheet_content.md`.
- Include R topics only if included because the user requested broad/full coverage.

Suggested format:

```text
# Topic Evidence Map

## Huffman coding / 霍夫曼编码
Priority: A
Detail mode: detailed

Evidence:
- Knowledge: K01 Page 18-24, Huffman coding procedure and prefix-free code discussion.
- Question: Q01 Page 4-5, coding/average length calculation question.

Used in cheatsheet:
- Procedure
- Expected length formula
- Prefix-free trap
```

Keep evidence entries concise. Do not quote long source passages. Use source IDs from `extraction_report.md` where possible. If page/slide markers are unavailable, cite the source file or source ID only. Do not include the evidence map inside printable HTML unless the user explicitly asks. `build_html.py` should not read `topic_evidence_map.md`.

## Cheatsheet Content Instructions

In `cheatsheet_content.md`, write compact markdown-like content for rendering. Use detail modes from `importance_ranking.md`.

Start with compact section headings such as `## Compression Calculations`; do not add a large course-title heading like `# COURSE_NAME Cheatsheet`, because printable space should prioritize exam content.

Supported block cues:

- `Topic: ...`
- `Formula: ...`
- `Def: ...`
- `Trap: ...`
- `Q: ...`
- `A: ...`
- `Compare: ...`
- `Procedure: ...`

Printable content hygiene rules:

- `Formula:` may only be used for real reusable formulas, equations, calculation shortcuts, legal element tests, statutory tests, compact rule templates, or exam-ready structured tests.
- Do not write ranking metadata inside printable content. Never include `Score`, `Scores`, `Lecture coverage`, `Exam evidence`, `Testability/formula value`, `Dependency`, `Error risk`, `Total`, `use Kxx`, or `question support Qxx` in any `Formula:`, `Def:`, `Procedure:`, `Trap:`, `Compare:`, `Q:`, or `A:` line.
- For law, policy, humanities, essay, or concept-heavy courses, prefer `Def:`, `Procedure:`, `Compare:`, `Trap:`, and `Q/A` over `Formula:` unless the material contains a real formula or reusable legal/statutory test.
- `importance_ranking.md` is a planning artifact. Its score lines, evidence summaries, source IDs, and ranking rationale must not be copied into `cheatsheet_content.md` as printable study content.

By default, generated `cheatsheet_content.md` should use concise bilingual topic labels in Topic lines:

```text
Topic: English topic / 中文标签 [Priority]
```

For example: `Topic: Entropy / 信息熵 [A]`, `Topic: Huffman coding / 霍夫曼编码 [A]`, and `Topic: Sampling / 采样 [A]`. The Chinese part is a short lookup label only. Do not translate long explanations unless requested. If the user explicitly requests English-only output, bilingual labels may be omitted.

Detailed topics should include as many of these as evidence supports: concise bilingual topic label, key definition, key formula, procedure/algorithm steps, comparison pair, trap/common error, exam-style Q/A if question evidence exists, and unit conversion or calculation shortcut if relevant.

Valid bilingual Topic headings should use real UTF-8 Chinese, for example `Topic: Raw bit rate and file size / 原始比特率与文件大小 [A]`, `Topic: Entropy / 信息熵 [A]`, and `Topic: Huffman coding / 霍夫曼编码 [A]`. If the user asks in Chinese or requests Chinese/bilingual output, keep bilingual Topic headings. Use compact English-first body text unless the user explicitly asks for full Chinese, and do not add long Chinese translations to every definition or procedure unless requested. Do not accept mojibake Topic labels such as `姣旂壒鐜囦笌...` as valid Chinese output.

Simple topics should include only: concise bilingual topic label, one short definition or core idea, one key formula if central, and one trap or compare item only if especially useful.

Omit topics should not appear in `cheatsheet_content.md` unless the user explicitly asks for a larger or more comprehensive sheet.

Also use headings, bullet lists, bold text, and simple markdown tables. Tables may be preserved as preformatted text by the renderer.

Keep bilingual topic labels short and lookup-oriented, for example `Topic: Quantization / 量化 [A]` and `Topic: Predictive coding / 预测编码 [B]`. The bilingual topic label is only a search aid; definitions, formulas, traps, comparisons, procedures, and Q/A must still be based on extracted evidence and must not invent unsupported technical content.

Do not embed long citations in the printable cheatsheet. Put traceability in `topic_evidence_map.md`.

## No-Fabrication Rules

- Never claim a topic will definitely appear in the exam.
- Never invent past-paper patterns.
- Never imply past-paper or sample-exam evidence exists when none was found in `questions/`.
- If no past papers or sample exams are found, use the exact Chinese caveat above.
- If materials are incomplete, label rankings and coverage as preliminary.
- If extraction failed or only filenames are available, label rankings, content coverage, and cheatsheet content as preliminary.
- Use extracted text whenever available; do not infer detailed document content from filenames alone.
- If a file in `questions/` has unclear source type, mark it uncertain and do not over-weight it.
- Do not over-weight question PDFs unless filename or extracted content confirms the source type.
- Prefer phrases like "high evidence", "appears in provided exam-style material", "testable", or "worth prioritizing" over predictive language.

## Build Commands

Check/install extraction dependencies:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/ensure_dependencies.py"
```

Extract materials:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/extract_materials.py" COURSE_NAME --root courses
```

Generate both HTML outputs:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/build_html.py" COURSE_NAME --root courses
```

Validate workflow artifacts and rendered HTML:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/validate_workflow.py" COURSE_NAME --root courses --mode full-auto --layout 3col --target-pages 1 --coverage-mode balanced-standard --detail-mode balanced
```

Render from a custom content file:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/build_html.py" COURSE_NAME --root courses --content path/to/cheatsheet_content.md
```

Build candidate units from extracted Markdown:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/build_candidate_units.py" COURSE_NAME --root courses
```

Show manual PDF export guidance:

```bash
SKILL_DIR="$HOME/.agents/skills/course-cheatsheet-maker"
python "$SKILL_DIR/scripts/export_pdf.py" courses/COURSE_NAME/outputs/cheatsheet_3col.html
```

## Final Review Checklist

- Course materials use only `materials/knowledge/` and `materials/questions/`.
- Normal course generation did not modify reusable skill scripts unless explicitly requested.
- `working/run_config.md` exists and records workflow mode, layout, integer target pages, coverage mode, detail mode, defaults, conflicts, and selection plan.
- Extraction dependencies were checked or installed automatically unless `COURSE_CHEATSHEET_NO_AUTO_INSTALL=1` was set.
- `working/extracted/` and `working/extraction_report.md` exist when materials include supported files.
- `working/candidate_units.jsonl` exists in Full Auto Mode and clearly distinguishes knowledge units from question evidence units.
- `working/knowledge_units.md` exists in Full Auto Mode and is based on `candidate_units.jsonl`.
- Extracted PDF/PPTX/DOCX/TXT/MD content preserves useful page, slide, document, or chunk locators.
- `extraction_report.md` includes source IDs and extracted unit counts.
- Extracted text was used whenever available.
- Filename-only analysis is clearly labeled preliminary and not ranked above C.
- `knowledge_units.md`, `topic_map.md`, `importance_ranking.md`, `topic_evidence_map.md`, and `cheatsheet_content.md` exist before HTML generation in Full Auto Mode.
- `importance_ranking.md` includes topic types, score-band-consistent priorities, evidence summaries, and recommended detail.
- `cheatsheet_content.md` follows `target_pages`, `coverage_mode`, and `detail_mode` without adding filler, and any page-budget repair changed this final printable file before HTML was rerendered.
- Question source types are inferred and uncertain files are marked.
- No unsupported exam predictions appear.
- The no-past-paper/sample-exam caveat appears when needed.
- Both 3-column and 4-column HTML files exist in Full Auto Mode.
- HTML is A4 landscape, compact, printable, and readable.
- `topic_evidence_map.md` is not embedded in printable HTML unless explicitly requested.
- Style reference content was not copied.
- `validate_workflow.py COURSE_NAME --root courses --mode full-auto --layout LAYOUT --target-pages TARGET_PAGES` passes before reporting Full Auto Mode complete. When actual print-page validation is requested, skipped print checks must be reported with the fallback reason.

---
> Source: [tsukiR1n/course-cheatsheet-maker](https://github.com/tsukiR1n/course-cheatsheet-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
