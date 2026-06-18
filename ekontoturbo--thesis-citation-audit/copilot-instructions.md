## thesis-citation-audit

> Use when someone asks to verify thesis citations, audit APA references, check thesis bibliography, weryfikacja cytowań pracy licencjackiej/magisterskiej, lub porównać tekst pracy ze źródłami PDF. Sentence-by-sentence verification of every APA citation in a thesis docx against source PDFs, with parallel sub-agents per subsection, and a consolidated markdown report. Supports Polish and English theses.


## What This Skill Does

Performs **sentence-by-sentence verification** of every APA citation in a thesis (.docx) against the actual source PDFs in a folder. Produces a master markdown report listing each citation with status (OK / wrong page / unsupported content / missing source / bibliography mismatch), plus an executive summary of critical problems and prioritized fixes.

This skill replicates a workflow that audited 4 versions of a real bachelor's thesis (~276 citations across 19 subsections) and caught dozens of fabricated page numbers, misattributed authors, source-file mismatches, and bibliography errors that a manual review would miss.

**Side effects:** writes ~20+ markdown files to a working directory + one final report. Dispatches many parallel sub-agents (real token cost). That's why `disable-model-invocation: true` — only the user can fire this.

## When to Use This Skill

- A student / author / reviewer needs every citation in a thesis checked against source PDFs.
- The thesis is in .docx format and sources are in a folder of PDFs (one folder per chapter is typical but not required).
- The user accepts that this takes 15–45 minutes wall-clock and burns tokens proportional to (# of subsections × avg PDFs per subsection).
- Languages: Polish, English, or mixed bilingual works.

## Required Inputs

The skill accepts three arguments (positional, all optional — if missing, ask the user):

1. **$1 — `docx_path`** — full path to the thesis docx, e.g. `C:\Users\name\Desktop\thesis_v10.docx`
2. **$2 — `sources_folder`** — full path to folder containing source PDFs (subfolders OK), e.g. `C:\Users\name\Desktop\Sources V2`
3. **$3 — `output_report_path`** — full path for the final consolidated report, e.g. `C:\Users\name\Desktop\CITATION_AUDIT_REPORT.md` (default: same folder as docx, named `CITATION_AUDIT_<docx_stem>.md`)

If any are missing, ask via `AskUserQuestion` before starting. Confirm all three paths exist before doing work.

## Dependencies

- **Python 3.11+** with `python-docx` (`pip install python-docx`). Check with `python -c "import docx"`.
- **`pdftotext`** (Poppler) on PATH — used by sub-agents to extract PDF text. Check with `which pdftotext` (Linux/macOS/Git Bash) or `where pdftotext` (PowerShell). On Windows install via `choco install poppler` or bundled with Git Bash + MSYS2.
- **Tools used:** `Agent` (general-purpose sub-agents, optionally `model: haiku`), `Bash`, `Write`, `Read`, `Edit`, `Glob`, `Grep`, `TaskCreate`/`TaskUpdate`.

If `python-docx` or `pdftotext` are missing, tell the user how to install and stop.

## Workflow

### Step 1 — Confirm task, plan in TaskCreate

Restate the task back to the user in 1–2 sentences, then create these tasks:

1. Extract docx and detect structure
2. Inventory citations + split into per-subsection files
3. Verify citations (parallel sub-agents, one per subsection)
4. Consolidate reports + executive summary

Mark task 1 `in_progress`. Use the tasks to track progress at each step.

### Step 2 — Extract docx and detect structure

Create a working directory: `<docx_folder>/_citation_audit/`. All intermediate files go there.

Run the extraction script:

```bash
python "C:\Users\erykc\.claude\skills\thesis-citation-audit\scripts\extract_thesis.py" "<docx_path>" "<workdir>"
```

The script writes:
- `<workdir>/thesis_full.txt` — every paragraph numbered `[P0000|Style] text`
- `<workdir>/structure.json` — detected chapters, subsections, bibliography, netografia ranges
- `<workdir>/citations.json` — every detected APA citation with paragraph number and surrounding context
- `<workdir>/bibliography.txt` — extracted bibliography + netografia

Read `structure.json`. Confirm to user: number of chapters, subsections, total citations. If the structure looks wrong (e.g. only 1 subsection detected on a multi-chapter thesis), the heading detection failed — ask the user to confirm the chapter structure manually or to tag headings in the docx as Heading 1/2/3 styles.

Mark task 1 completed, task 2 in_progress.

### Step 3 — Split into per-subsection files

```bash
python "C:\Users\erykc\.claude\skills\thesis-citation-audit\scripts\split_sections.py" "<workdir>"
```

This reads `structure.json` + `citations.json` and writes:
- `<workdir>/sections/section_<id>.txt` — one file per subsection, with paragraphs that have citations
- `<workdir>/sections/_index.json` — list of sections with citation counts + source folder hints

Mark task 2 completed, task 3 in_progress.

### Step 4 — Map sources folder

Glob the sources folder and produce a list of (subsection_hint → folder_path). Two common layouts:

- **Per-chapter subfolders** like `Sources/Chapter 1/1.1/` — common in Polish theses. Match subsection `1.1` to folder `Chapter 1/1.1/`.
- **Flat folder** — all PDFs in one folder. Sub-agents will glob the whole folder for each citation.

If the folder uses non-ASCII characters in path names (Polish: `Źródła`, German umlauts, etc.) test that `pdftotext` can open at least one file. If it can't, tell sub-agents to copy each PDF to `C:\tmp\` (or equivalent) before extracting.

**Netografia handling:** netografia entries (web sources) are usually NOT in the source folder. They live either in:
- A separate `Netografia.txt` file inside the chapter folder, or
- A sub-section of the bibliography (look for the "Netografia" heading in `bibliography.txt`).

Pass the bibliography path to every sub-agent so they can match netografia citations there.

### Step 5 — Dispatch parallel sub-agents

For each subsection in `_index.json`, dispatch a `general-purpose` sub-agent with the template prompt from `templates/section_prompt.md`. Use `model: haiku` if the model frontmatter allows (cheap+fast for per-citation verification); fall back to default model if not.

**Fire ALL sub-agents in parallel** (multiple `Agent` tool uses in a single message) up to the dispatcher's concurrent limit (~6–12 at a time works). If there are 15+ subsections, fire in waves of 6 each.

Each sub-agent gets:
- `section_id` (e.g. "1.1")
- Path to its section file (`section_<id>.txt`)
- Path to its source folder (best match from step 4)
- Path to the bibliography file
- Path to write its report (`<workdir>/reports/section_<id>_report.md`)
- Concrete instructions to: verify EVERY citation, open the actual PDF, distinguish PDF page from printed page, mark status per the legend, and never hallucinate.

See `templates/section_prompt.md` for the exact prompt template. Substitute `{{section_id}}`, `{{section_file}}`, `{{sources_folder}}`, `{{bibliography_file}}`, `{{report_path}}` before sending.

Run sub-agents in background (`run_in_background: true`) so progress notifications come in async. As each completes, mark its row in TaskCreate.

### Step 6 — Consolidate reports + executive summary

Once all sub-agent reports exist in `<workdir>/reports/`:

```bash
python "C:\Users\erykc\.claude\skills\thesis-citation-audit\scripts\merge_report.py" "<workdir>" "<output_report_path>"
```

This concatenates all per-section reports into one big markdown file, ordered by section ID, with a header block.

Then YOU (the orchestrator) write an **executive summary** that prepends the final report. Use `templates/exec_summary.md` as a structure guide. Walk through every sub-agent's reported issues and group them into:

1. **🚨 Critical (orphaned citations, fabricated pages, wrong author)** — anything a reviewer will catch in 2 minutes.
2. **🔴 Bibliography ↔ text mismatches** (wrong year, wrong edition, missing co-author).
3. **❌ Files don't match declared work** (PDF is a review instead of the book; printed s. X doesn't exist).
4. **⚠️ Page-number drift** (off by a few pages, often a pattern: e.g. all Author X citations −1 page).
5. **Patterns** — systematic issues that repeat across chapters.
6. **Recommendations** — prioritized fix list, easiest wins first.

Prepend the executive summary to the merged report (use `Edit` on the output file or rewrite via Python).

Mark task 4 completed.

### Step 7 — Report results to user

Brief summary in chat:
- Path to the final report
- Headline numbers: total citations checked, % OK, count of critical issues
- Top 3 most urgent fixes
- Reminder that detailed per-citation verifications are in the report

## Output Format

The final report has this structure:

```markdown
# CITATION AUDIT REPORT — <thesis stem>

**Source docx:** <path>
**Sources folder:** <path>
**Date:** YYYY-MM-DD
**Scope:** N citations across M subsections

**Legend:**
- ✅ OK — content + page correct
- ⚠️ STRONA — wrong page number (content is in source but on a different page)
- ❌ TREŚĆ — claim not supported by cited fragment / over-interpretation
- ❓ BRAK ŹRÓDŁA — source file unavailable / unverifiable
- 🔴 BIBLIOGRAFIA — citation ↔ bibliography mismatch (year, authors, edition, pagination)

---

## EXECUTIVE SUMMARY
[critical issues, patterns, recommendations]

---

# Section <id> — <title>

## [P0XXX] (citation)
**Sentence:** "..."
**Source:** filename
**PDF page:** X (printed s. Y)
**Status:** ✅ / ⚠️ / ❌ / ❓ / 🔴
**Comment:** ...

[... etc per citation, per section]
```

## Notes & Guardrails

- **NEVER hallucinate.** If a sub-agent cannot open a PDF, it must say so and mark ❓. Never invent page numbers or content.
- **PDF page ≠ printed page.** Almost always there's an offset (preface adds ~10 pages). The sub-agent should verify offset once per source by spotting a printed page number in the header/footer, then apply it consistently.
- **Don't trust file names.** PDFs are often misnamed (a PDF labeled "Author 2025" can be a 2022 paper). The sub-agent must verify year/title/DOI inside the PDF, not from the filename.
- **Don't trust the bibliography blindly.** The reverse also happens: bibliography says "Journal A 451–474" but file is "Journal B 1034–1046". Flag as 🔴.
- **Watch for preprint-vs-publication paginations.** Common in academic articles. Bibliography cites published pages (e.g. 528–544) but author cites preprint pages (s. 1–14). Flag as ⚠️ with the printed-page mapping.
- **Sentence-by-sentence verification.** A page citation can be "in range" yet the specific claim isn't on the cited page. Always read enough of the PDF to confirm the exact claim, not just that the topic appears nearby.
- **Don't do the executive summary on auto-pilot.** Read each section report carefully, look for patterns (e.g. every Heilman citation has same issue → systematic error). Use the patterns block in the summary.
- **No model invocation** — `disable-model-invocation: true` is set. Users must explicitly fire this with `/thesis-citation-audit`.
- **One-shot per session.** This skill is fire-and-forget. The user runs it, gets a report, decides what to fix manually.
- **Cost discipline.** If user mentions a 100-page paper with 5 citations, do NOT spawn 19 sub-agents — collapse to 1 or 2 sub-agents. Tune sub-agent count to (# subsections × avg citations).

## Supporting Files

- `scripts/extract_thesis.py` — docx → text + structure + citations
- `scripts/split_sections.py` — per-subsection file generation
- `scripts/merge_report.py` — consolidate reports into one file
- `templates/section_prompt.md` — exact prompt template for sub-agents
- `templates/exec_summary.md` — structure template for the executive summary
- `README.md` — install + usage instructions for end users

---
> Source: [ekontoTURBO/thesis-citation-audit](https://github.com/ekontoTURBO/thesis-citation-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
