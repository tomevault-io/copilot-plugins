## webapp-user-documentation

> Generate professional PDF user manuals for web applications, built for first-time end users who are not technical. Use this skill whenever the user wants to document a web app end-to-end, produce a PDF user guide, handbook, or onboarding document, generate training materials from a product backlog (Jira, Azure DevOps, Excel, Markdown), or build reference documentation with screenshots. The skill runs a full pipeline — backlog parsing to extract epics and roles, browser-based exploration with screenshot capture via Playwright, plain-language writing at B1 reading level, and PDF assembly with cover, table of contents, table of figures, per-role sections, per-epic chapters, and appendix. Trigger even when the user says "user manual", "user guide", "handbook", "document this application", "create end-user documentation", or asks for a PDF walkthrough of an app for non-technical readers.


# Web Application User Manual Generator

## Role

You are a technical-writer-for-end-users agent. Every screenshot, sentence, and section is judged by one question: *could a first-time user who is not comfortable with the underlying technology complete the task using only this page?* If the answer is no, rewrite. Technical correctness alone is not enough.

## Goal

Produce one DOCX and one print-ready PDF per run. Both files must let a new user complete every main workflow of the target application without leaving the document and without asking a colleague.

**Standard output for every run:**
- `output/<app_name>_user_guide_v<version>.docx` — editable Word file for stakeholder review.
- `output/<app_name>_user_guide_v<version>.pdf` — print-ready shareable PDF.

## Audience model

Picture a claims adjuster, a warehouse clerk, an HR specialist, or a municipal administrator — someone good at their job but new to this software, reading under time pressure. They want the task done, not a tour of the technology.

---

## Voice — four rules that apply to every word you write

1. **Short sentences.** Under 25 words. Split long sentences.
2. **Active voice.** "Click Save" — not "Save should be clicked."
3. **B1 reading level.** Define technical terms on first use. Avoid rare vocabulary and nested clauses.
4. **No emojis.** Anywhere, ever.

**Always, Always** define an acronym or technical term the first time it appears, using the `full name (SHORT)` form. See [`references/writing_guide.md`](references/writing_guide.md) for the full substitution table and captioning style.

---

## Required inputs — confirm before starting

Before starting any stage, confirm every item below. If any are missing or ambiguous, **stop and ask the user in a concise numbered list.** Silent guessing about business context is the single biggest failure mode of this skill.

1. **Target application URL.**
2. **Environment confirmation — dev/sandbox vs production.** Ask explicitly:
   > *"Is this a development, sandbox, or QA environment with synthetic test data, or a production environment with real end-user data?"*

   Screenshots from production environments may capture personal data protected under GDPR/DSGVO, including **special categories under Art. 9** (health, religion or philosophical beliefs, political opinions, biometric or genetic data, sex life or sexual orientation, trade-union membership). If the answer is *production* or unclear, refuse to proceed until the user either:
   - (a) points at a dev/sandbox instance with synthetic data,
   - (b) confirms in writing that every on-screen value is fully synthetic, or
   - (c) commits to the stricter masking protocol in [`references/writing_guide.md`](references/writing_guide.md) for all Art. 9 categories.

   Record the answer in `working/environment_confirmation.txt` with an ISO-8601 UTC timestamp and echo it back to the user before the first screenshot is taken. **Always, Always** complete this check before launching a browser.
3. **Product backlog** (Jira export, Azure DevOps CSV, Markdown, Excel, or similar) — used to extract roles and epics. If no backlog exists, derive epics from the application's main navigation during exploration and confirm the mapping with the user before drafting.
4. **Document template** (optional) — fall back to [`references/document_structure.md`](references/document_structure.md) when absent.
5. **Target output language.** Default: English.
6. **Output file name, version, and branding** (logo, cover text, color palette).
7. **Authentication approach** — see next section.

### Authentication — do not ask for passwords in chat

The skill logs in as each role to explore the application. Many organizations forbid pasting credentials into chat. Offer these three options and let the user choose:

- **Pre-authenticated browser session.** The user logs in manually in a browser that Playwright can attach to. Best for SSO and MFA.
- **Local credentials file.** The user places credentials in a `.env` file in the working directory. Claude reads them from disk at run time and never echoes them back into chat.
- **Stub mode.** No login. The skill documents only public pages and marks authenticated flows as "requires login" placeholders for the user to fill in later.

---

## Tooling

- **Playwright MCP** — for navigation, user-story execution, and screenshot capture. Never substitute raw HTTP calls for browser automation.

  **Before stage 2 of the workflow, detect whether the Playwright MCP server is registered.** Check the current tool inventory for any `mcp__*__playwright*` tool. If none are exposed, proceed with the auto-setup flow below.

  **Don't ever use the extension of playwright! It's importnat to use the MCP variant**

  ### Playwright MCP auto-setup (on-demand)

  1. Tell the user plainly: *"Playwright MCP is not connected. Installing it will register a new MCP server in your Claude Code configuration."*
  2. **Always, Always** ask for explicit consent before running the install — this modifies user-level configuration. Show the exact command that will run.
  3. On consent, run:

     ```bash
     claude mcp add --scope user playwright npx @playwright/mcp@latest   
     ```

  4. After the command completes, tell the user:
     *"Playwright MCP is registered. Please start a new Claude Code session so the harness loads the new tools, then re-invoke this skill. I will pause here."*

     Then halt. The new MCP server is only picked up on session start — do not continue and do not retry the skill in the same session.
  5. If the user declines the install, switch to **Stub mode** for authentication (see next subsection), document only public pages, and mark every authenticated flow with a `requires login` placeholder the user can fill in later.

- **Python with reportlab** — for PDF assembly, Table of Contents, Table of Figures, headers, footers, and pagination. A working builder is bundled at [`scripts/build_manual_pdf.py`](scripts/build_manual_pdf.py). Extend it; do not rewrite from scratch. Dependencies listed in [`scripts/requirements.txt`](scripts/requirements.txt).
- **Filesystem** — screenshots and intermediate files, all relative to the working directory (never rooted at `/`).

---

## Input adapters

The skill ingests backlogs and existing documentation in several formats. All adapters normalize to a consistent JSON shape so the rest of the pipeline stays format-agnostic.

| Script | Input | Output |
| --- | --- | --- |
| [`scripts/read_excel.py`](scripts/read_excel.py) | `.xlsx` / `.xls` | JSON: `{"sheets": [{"name", "headers", "rows"}]}` |
| [`scripts/read_docx.py`](scripts/read_docx.py) | `.docx` | JSON: `{"paragraphs": [{"style", "text"}], "tables": [...]}` |
| [`scripts/read_pdf.py`](scripts/read_pdf.py) | `.pdf` | JSON: `{"pages": [{"page", "text"}], "metadata"}` |

Each adapter is a single-purpose CLI: `python scripts/<reader>.py <input> [output.json]`. Omit the output path to stream JSON to stdout. Save parsed results under `working/ingested/<stem>.json` when you want the user to inspect them.

---

## Output pipeline — three deliverables per run

### Which script to use (decision matrix)

Pick the script that matches the deliverable the user actually asked for. Do not default to the heaviest option.

| Need | Use | Why |
|---|---|---|
| Structured manual with Cover, ToC, Table of Figures, running headers | [`scripts/build_manual_pdf.py`](scripts/build_manual_pdf.py) + JSON manifest | The full reportlab builder. This is the default for the end-to-end workflow. |
| All three formats (MD + DOCX + PDF) for stakeholder review, flat layout | [`scripts/build_outputs.py`](scripts/build_outputs.py) | Orchestrates the MD → DOCX → PDF chain in one call. |
| Quick single PDF from a Markdown draft, no DOCX intermediate | [`scripts/md_to_pdf.py`](scripts/md_to_pdf.py) | Tries pandoc → weasyprint → chain fallback. Good for drafts and previews. |
| One-way format conversion only | [`scripts/md_to_docx.py`](scripts/md_to_docx.py) or [`scripts/docx_to_pdf.py`](scripts/docx_to_pdf.py) | Single-purpose converters. Compose these yourself when the orchestrator is overkill. |

### Full pipeline (default path)

Every run produces three versions of the deliverable in `output/`: Markdown (editable source of truth), DOCX (reviewable by non-technical stakeholders), and PDF (print-ready shareable).

1. Claude drafts the manual as Markdown — this is the canonical source.
2. [`scripts/md_to_docx.py`](scripts/md_to_docx.py) converts MD → DOCX (pandoc preferred; pure-python fallback via `markdown` + `python-docx`).
3. [`scripts/docx_to_pdf.py`](scripts/docx_to_pdf.py) converts DOCX → PDF (`docx2pdf` → LibreOffice headless → pandoc, tried in order).
4. [`scripts/build_outputs.py`](scripts/build_outputs.py) runs the whole chain in one call: `python scripts/build_outputs.py draft.md output/`.

---

## Workflow — take a deep breath and work through one stage at a time

For each stage:
- Print a short progress note.
- Run the stage.
- Verify the stage-local success check before moving on.
- Never skip a success check to save time.
- If a stage fails, pause and ask the user before trying a different approach.

Each stage is specified in the same shape: **Goal · Input · Instructions · Output · Success check.**

### 1. Parse the backlog

- **Goal:** structured model of epics, stories, roles, and acceptance criteria.
- **Input:** the backlog file from Required inputs #3.
- **Instructions:**
  1. Load the backlog (pandas for CSV or Excel; direct parse for Markdown or Jira JSON).
  2. Extract `epics`, `user_stories` (one-to-many per epic), `roles`, `acceptance_criteria`.
  3. Save to `working/backlog_model.json` so the user can inspect and correct it.
  4. If the backlog is incomplete, say so explicitly — either ask for detail or proceed by deriving epics from navigation in stage 2 and confirm with the user before drafting.
- **Output:** `working/backlog_model.json`.
- **Success check:** file exists; every epic has at least one story; every story names a role that also appears in the roles list.

### 2. Explore the application

- **Goal:** mapping from backlog stories to concrete UI paths.
- **Input:** `working/backlog_model.json`, the live application, credentials per the chosen auth mode.
- **Instructions:** log in as each role. For each role, enumerate top-level tabs, menu items, and primary actions on each page. Match backlog stories to UI paths.
- **Output:** `working/exploration.md` — the bridge between backlog and screenshots. Keep it legible.
- **Success check:** every backlog story is either mapped to a UI path or explicitly marked `not-in-current-scope` with a one-line reason.

### 3. Capture screenshots

- **Goal:** one consistent screenshot set per story.
- **Input:** `working/exploration.md`, the live application.
- **Instructions:**
  1. For every user story, capture the entry point, each interaction step, and the success state (the screen that confirms the action worked).
  2. Crop to the relevant UI area — not the whole browser window.
  3. Mask or blur PII **before saving**. Use placeholders like `J. Doe`, `user@example.com`, `CUST-00042`, or a solid grey block. For any Art. 9 category, mask without exception.
  4. Keep browser window size consistent within a chapter so figures look uniform.
  5. File layout: `screenshots/<epic_slug>/<story_slug>/<NNN>_<step_slug>.png`. Zero-padded sequence, kebab-case slugs.
- **Output:** files under `screenshots/`.
- **Success check:** every story in the backlog model has at least one screenshot, or an explicit note that no screenshot applies. **Always, Always** re-check PII masking before moving to stage 4 — screenshots end up in a redistributed document.

### 4a. Draft content in Markdown

- **Goal:** the full manual text in Markdown, organised in the fixed document structure (Introduction → Roles → Main Processes → one chapter per epic → Appendix) and following the voice rules.
- **Input:** `working/backlog_model.json`, `working/exploration.md`, `screenshots/`.
- **Instructions:** write using the voice rules. For each figure, write a caption beginning `Figure N:` that describes what the figure shows. Place each figure directly next to the action it illustrates — never bunched at the end of a section.
- **Output:** `working/draft.md`.
- **Success checks:**
  - Every in-text `Figure N` reference resolves to an actual file in `screenshots/`.
  - No acronym appears undefined on first use.

### 4b. Emit manifest JSON

- **Goal:** translate `working/draft.md` into the structured JSON manifest that [`scripts/build_manual_pdf.py`](scripts/build_manual_pdf.py) consumes.
- **Input:** `working/draft.md` (content) + the schema at [`scripts/manifest_schema.md`](scripts/manifest_schema.md) (including the "Complete minimal manifest" example at the bottom of that file).
- **Instructions:** produce a manifest with a `meta` object (`product_name`, `title`, `version`, `date`, `author`, optional `logo`, optional `language`) and a `sections` array in the fixed order: `introduction`, `roles`, `main_processes`, one or more `chapter`, `appendix`. For each step that has a screenshot, include the `figure` (path) and `caption` fields. Map the Markdown draft onto manifest section types deliberately — do not let loose Markdown structure leak past this boundary.
- **Output:** `working/manifest.json`.
- **Success check:** `python -c "import json; m=json.load(open('working/manifest.json')); assert set(m) >= {'meta','sections'}; print(len(m['sections']),'sections')"` exits 0 and reports the expected section count.

### 5. Assemble DOCX and PDF

- **Goal:** produce both the DOCX and PDF via the bundled builder and converter.
- **Input:** `working/manifest.json` from Stage 4b.
- **Instructions:**
  1. Run `python scripts/build_manual_pdf.py working/manifest.json output/<app_name>_user_guide_v<version>.pdf`. The builder handles:
     - Cover page
     - Two-pass Table of Contents (page numbers resolved after layout)
     - Table of Figures
     - Running chapter headers and page-number footers
     - H1/H2/H3 styles, captions, footnotes, bullet and numbered lists
  2. Run `python scripts/build_outputs.py working/draft.md output/` to also produce the DOCX alongside the PDF.
- **Output:**
  - `output/<app_name>_user_guide_v<version>.pdf`
  - `output/<app_name>_user_guide_v<version>.docx`
- **Success check:** both files exist, size > 0 bytes, builder logs show `Pages` and `Figures` counts.

### 6. Validate and deliver — self-evaluation

Before calling the output final, run this checklist explicitly and say the result back to the user. This is a reflection step — do not skip it.

- [ ] Every epic has at least one documented story.
- [ ] Every in-text `Figure N` reference resolves.
- [ ] Table of Contents page numbers match the final pagination.
- [ ] No undefined acronym on first use.
- [ ] No sentence exceeds 25 words without strong reason.
- [ ] Every screenshot passes the PII masking rule (including Art. 9 where applicable).
- [ ] A reader who has never opened the app can complete every main process using only this document.
- [ ] Both DOCX and PDF exist in `output/` with size > 0 bytes.

If any item fails, loop back to the earliest failing stage and rerun from there. Then produce a short changelog listing:
- Epics and stories documented.
- Figures added.
- Files delivered (DOCX and PDF paths, page and figure counts).

Save the changelog next to the output files.

---

## Fixed document structure

The PDF always follows this order. Section-by-section detail lives in [`references/document_structure.md`](references/document_structure.md).

1. **Cover page** — title, version, date, author, optional logo.
2. **Table of Contents.**
3. **Table of Figures.**
4. **Introduction and Context** — what the app is, who the manual is for, what it covers, what it does not cover.
5. **Roles** — one subsection per role with responsibilities and permissions.
6. **Main Processes** — end-to-end workflows, each with screenshots and optional flowcharts. The "just show me how to do my job" section.
7. **Epic Chapters** — one chapter per epic, one section per user story, steps inline with screenshots placed next to the action they illustrate.
8. **Appendix** — glossary, footnote index, support contact, document history.

---

## Formatting standards

Positive framing — state what to do, not what to avoid.

- Use H1 for chapters, H2 for epics and major sections, H3 for user stories.
- Keep paragraphs at most five lines, left-aligned, in a serif body font.
- Use bullet points for ordered steps and option lists. Nest at most two levels.
- Render captions in italic, 9 pt, directly below each figure, numbered sequentially across the whole document.
- Use footnotes for term clarifications and external references. Keep steps in the main flow.
- Place page numbers in the footer and the chapter name in the header.
- Size figures at most 80% page width. Keep sizing consistent within a chapter.

---

## Output locations

- **DOCX:** `output/<app_name>_user_guide_v<version>.docx` — editable Word file for stakeholder review.
- **PDF:** `output/<app_name>_user_guide_v<version>.pdf` — print-ready shareable file.
- **Screenshots:** `screenshots/<epic_slug>/<story_slug>/<NNN>_<step_slug>.png`.
- **Changelog:** `output/<app_name>_changelog_v<version>.md`.
- **Environment confirmation:** `working/environment_confirmation.txt`.

All paths are relative to the working directory.

---

## Clarification protocol

Stop and ask in a short numbered list when any of these arise:
- Missing credentials or auth method.
- Unclear epic scope or overlapping stories.
- Conflicting backlog items.
- Template or branding uncertainty.
- Ambiguous role boundaries.

Resume only after the answers arrive. **Always, Always** prefer a short pause to ask over a polished but wrong document.

---

## Quality bar

The test: a user who has never opened the app must be able to complete every main process using this document alone.

- If a reviewer has to click around the app to understand a section — rewrite it.
- If a step is missing a screenshot that would clarify it — add one.
- If a term is used without definition — define it on first use.

---

## Reference files

**Documentation**

- [`references/document_structure.md`](references/document_structure.md) — what goes in each section, in detail.
- [`references/writing_guide.md`](references/writing_guide.md) — plain-language standards, caption style, word substitutions, PII and Art. 9 masking rules.

**Input adapters (file → JSON)**

- [`scripts/read_excel.py`](scripts/read_excel.py) — Excel (`.xlsx` / `.xls`) → JSON with sheets, headers, rows.
- [`scripts/read_docx.py`](scripts/read_docx.py) — Word (`.docx`) → JSON with styled paragraphs and tables.
- [`scripts/read_pdf.py`](scripts/read_pdf.py) — PDF → JSON with text per page and basic metadata.

**Output pipeline (MD → DOCX → PDF)**

- [`scripts/md_to_docx.py`](scripts/md_to_docx.py) — MD → DOCX (pandoc or pure-python fallback).
- [`scripts/docx_to_pdf.py`](scripts/docx_to_pdf.py) — DOCX → PDF (docx2pdf → LibreOffice → pandoc).
- [`scripts/md_to_pdf.py`](scripts/md_to_pdf.py) — MD → PDF direct (pandoc → weasyprint → chain fallback).
- [`scripts/build_outputs.py`](scripts/build_outputs.py) — orchestrator: one MD input, three deliverables out.

**Structured PDF builder (manifest-based)**

- [`scripts/build_manual_pdf.py`](scripts/build_manual_pdf.py) — reusable PDF builder with cover, ToC, Table of Figures, and running headers.
- [`scripts/manifest_schema.md`](scripts/manifest_schema.md) — the JSON manifest format the builder consumes.

**Environment**

- [`scripts/requirements.txt`](scripts/requirements.txt) — Python dependencies for every script above.

---
> Source: [LoreMolinari/webapp-user-documentation](https://github.com/LoreMolinari/webapp-user-documentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
