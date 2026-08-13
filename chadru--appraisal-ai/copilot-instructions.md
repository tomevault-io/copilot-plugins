## appraisal-ai

> When a session begins:

# Claude Code Instructions for This Repo

## On Every Session Start

When a session begins:

### 1. Load context snapshot
Read `.claude/CPM_SNAPSHOT.md` if it exists. This contains project context, architecture decisions, known issues, and lessons learned from previous sessions. **This is your institutional memory — read it before doing anything else.**

### 2. Check virtual environment

```bash
ls ~/appraisal_ai/venv/bin/python
```

- **If it exists:** The user has already set up. Use `~/appraisal_ai/venv/bin/python` for all script commands. Tell the user:
  > Welcome back. Your virtual environment is set up and ready. What are we working on?

- **If it doesn't exist:** The user is either new or hasn't set up yet. Check if `requirements.txt` dependencies are needed and walk them through setup (see Step 3 below).

### 3. CPM snapshot updates (automatic — do not wait for user to ask)

Update `.claude/CPM_SNAPSHOT.md` at these trigger points:

- **After delivering a draft/grid** (end of Phase 5) — record pipeline results, issues found, project status
- **After user provides corrections** (Phase 4.5 Step 4) — record what was corrected and whether it reveals a pattern worth adding to Lessons Learned
- **When user says "run CPM" / "do a CPM pass"** — full refresh of the entire snapshot
- **End of any session with significant work** — capture new rules, workflow changes, lessons

What to include: project status, lessons learned summary, architecture changes, known issues.
What NOT to include: client-specific data (addresses, dollar amounts, names) — the snapshot is in git.

## New User Onboarding

When a user first opens this repo and asks for help (e.g., "help me get started", "how do I use this", "I just cloned this repo"), walk them through setup step by step:

### Step 1: Ask for their project folder

Say something like:

> I'll help you get set up. First, I need you to create a **new empty folder** for your project. Name it with your file number and address, like `2026-001 123 Main St`.
>
> **Important:** The folder needs to be empty so I can set it up correctly. If you already have documents for this project, that's fine — I'll tell you where to put them in a moment.
>
> Once you've created the folder, I need the folder path. The easiest way is to **drag the folder from File Explorer (Windows) or Finder (Mac) directly into this terminal window** — it will paste the full path automatically.
>
> Or you can copy the path manually:
> - **Windows:** Open File Explorer, navigate to the folder, click the **address bar** at the top — it will highlight the full path. Right-click and **Copy**, then paste it here.
> - **Mac:** Right-click the folder in Finder, hold **Option**, and click **"Copy as Pathname"**. Then paste it here.

### Step 2: Create the project folder structure

Once they give you the path, create these subfolders inside it. **Do not use dots, spaces, or numbered prefixes in folder names** — just plain names:

```
<project-folder>/
├── Subject/         — engagement letter, assessor card, deed, LOI, CoStar, etc.
├── Narrative/       — output drafts will go here
├── Comparables/     — comp sheets, CoStar/MLS printouts, sale data
├── Exhibits/        — maps, photos, figures
└── Template/        — put your completed appraisal template (.docx) and grid (.xlsx) here
```

Create all five folders using mkdir. Then tell the user:

> I've set up your project folder with five subfolders. Now you need to add your files:
>
> 1. **Subject/** — Drop in your engagement letter, assessor card, deed, LOI, CoStar reports, and any other subject property documents.
> 2. **Comparables/** — Drop in your comp sheets, CoStar/MLS printouts, and sale data.
> 3. **Exhibits/** — Drop in any maps, photos, or figures.
> 4. **Template/** — Drop in a **completed appraisal** (.docx) that you want to use as your master template. If you have a grid/schedule (.xlsx), drop that in too.
>
> Go ahead and add your files, then come back and tell me when you're ready.

### Step 3: Set up virtual environment and install dependencies

**Always use a virtual environment.** Never install packages directly on the host system.

Create the venv and install packages:

```bash
python3 -m venv ~/appraisal_ai/venv
~/appraisal_ai/venv/bin/pip install -r ~/appraisal_ai/requirements.txt
```

If the user is on **Windows (not WSL)** and using PowerShell:
```powershell
python -m venv ~\appraisal_ai\venv
~\appraisal_ai\venv\Scripts\pip install -r ~\appraisal_ai\requirements.txt
```

**Running scripts:** Always use the venv's Python interpreter directly. Shell state doesn't persist between commands in Claude Code, so `source activate` won't stick. Use the full path:

```bash
~/appraisal_ai/venv/bin/python -c "import sys, os; sys.path.insert(0, os.path.expanduser('~/appraisal_ai/scripts')); ..."
```

Tell the user:

> I've created a virtual environment so all the Python packages stay isolated from your system. I'll use it automatically when running scripts — you don't need to do anything extra.
>
> If you ever want to run scripts manually outside of Claude Code, activate the venv first:
>
> **WSL/Linux/Mac:** `source ~/appraisal_ai/venv/bin/activate`
> **Windows PowerShell:** `~\appraisal_ai\venv\Scripts\Activate.ps1`
>
> You'll know it's active when you see `(venv)` at the start of your terminal prompt.

### Step 4: Scan the project documents

Claude reads all project documents directly — no external OCR tools needed. Claude can natively read PDFs (text and scanned), images, and all common document formats.

Here's the workflow:

1. List all files in Subject/, Comparables/, Exhibits/, and the project root
2. Read each PDF directly using the Read tool (handles both text and scanned PDFs)
3. Read each .docx using `extract_docx_text()` from `scripts/utils.py`
4. Read each .xlsx using `extract_xlsx_text()` from `scripts/utils.py`
5. Review visual documents (maps, surveys, photos) by reading them as images
6. Extract all structured fields from the content
7. Write `data.md` using `save_md()` from `scripts/utils.py`

Walk through every `*** UPDATE ***` and `*** VERIFY ***` field with the user **one group at a time**. Don't just dump a list — ask about each group of related fields, explain what the field is, suggest values when you can infer them from documents, and help the user fill in as many as possible. Group fields logically (dates, property details, value conclusions, comp analysis). For fields the user doesn't have yet, confirm they want to leave them as `*** UPDATE ***` and move on. Re-read documents if something was missed.

### Step 5: Build the draft and grid

Once data.md is filled in:
- Build the narrative draft with tracked changes
- Build the grid/schedule
- Tell them where the output files are and how to open them in Word/Excel

### Step 6: Explain the output

> Open the DRAFT .docx file in Word. You'll see tracked changes — strikethrough text is the old template value, underlined text is your new project value. Go through each one and Accept or Reject. The .xlsx grid has your subject column updated — open it in Excel to verify.

## Reading Large PDFs

Some PDFs are too large for the Read tool to handle in one pass. Use this fallback workflow:

1. **Before reading any PDF**, check its file size:
   ```bash
   ls -lh /path/to/file.pdf
   ```

2. **If the file is over ~10 MB, or if the Read tool errors out**, split it first:
   ```bash
   ~/appraisal_ai/venv/bin/python -c "
   import sys, os; sys.path.insert(0, os.path.expanduser('~/appraisal_ai/scripts'))
   from utils import split_pdf
   chunks, chunk_dir = split_pdf('/path/to/file.pdf')
   for c in chunks:
       print(c)
   print('CHUNK_DIR=' + chunk_dir)
   "
   ```

3. **Read each chunk** separately using the Read tool, then combine the content.

4. **Clean up** the temporary chunk files when done:
   ```bash
   ~/appraisal_ai/venv/bin/python -c "
   import sys, os; sys.path.insert(0, os.path.expanduser('~/appraisal_ai/scripts'))
   from utils import cleanup_pdf_chunks
   cleanup_pdf_chunks('/path/to/_pdf_chunks_filename')
   "
   ```

The `split_pdf()` function splits the PDF into 5-page chunks by default. You can pass a different `pages_per_chunk` value if needed. The chunks are written to a `_pdf_chunks_<name>/` directory next to the original file.

## Reading .docx Files

**Never use the Read tool to open .docx files.** The Read tool cannot read binary .docx files and will error. Instead, always use the `extract_docx_text()` function from `scripts/utils.py`:

```python
~/appraisal_ai/venv/bin/python -c "
import sys; sys.path.insert(0, 'scripts')
from utils import extract_docx_text
text = extract_docx_text('/path/to/file.docx')
print(text)
"
```

This works because .docx files are ZIP archives containing XML. Our utility opens the ZIP, parses `word/document.xml` with lxml, and extracts all text nodes.

## Mandatory Agent Workflow

**When the user asks to draft a report, build a grid, run the pipeline, or review outputs, you MUST use the agent team. Do not do the work manually in the main conversation.**

### Trigger phrases → Required action

| User says | You do |
|-----------|--------|
| "draft the report", "build the narrative", "redraft" | Run `/draft` skill — dispatches **Draft Builder** agent |
| "build the grid", "update the grid" | Run `/grid` skill — dispatches **Grid Builder** agent |
| "run the pipeline", "run everything", "start from scratch" | Run `/run` skill — dispatches **all agents** across 5 phases |
| "review the draft", "QA check", "check for errors" | Run `/review` skill — dispatches **QA Reviewer** + **USPAP Reviewer** agents |

### How it works

The `/run` pipeline dispatches 10 agents across 5 phases:

1. **Phase 1 — Read** (3 agents in parallel): Subject Reader, Comp Reader, Exhibit Reader
2. **Phase 2 — Extract** (2 agents in parallel): Field Extractor, Comp Grid Agent → then **pause for user review** of `*** UPDATE ***` / `*** VERIFY ***` fields
3. **Phase 3 — Build** (3 steps): Comp Writer first → then Draft Builder + Grid Builder in parallel. Comp Writer generates `comp_writeups.md` which the Draft Builder reads to replace LAND SALE NO. sections.
4. **Phase 4 — Review** (2 agents in parallel): QA Reviewer, USPAP Reviewer → auto-fix loop (max 2 iterations)
5. **Phase 5 — Final Review (BLOCKS DELIVERY)**: Orchestrator performs the full Pre-Delivery Review checklist (see below). **You cannot hand files to the user until Phase 5 passes.**

### Phase 5 — Final Review (mandatory, non-skippable)

This is the last gate. The orchestrator (you) does this directly — not an agent. After Phase 4 completes:

1. **Extract narrative text** using `extract_docx_text()` and compare every data.md field value against it. Flag missing replacements and stale template values.
2. **Open the grid** with openpyxl and verify: correct sheet, subject column matches data.md, comp columns match comp_grid.md, formulas intact, adjustment rows preserved.
3. **Cross-check grid ↔ narrative**: comp summary table PSFs match grid adjusted PSFs, subject data identical in both, comp numbers and order consistent.
4. **Report findings** to the user with the output files. List anything that needs their attention.

**If Phase 5 finds errors:** Fix them before delivering. If you can't fix them (ambiguous data, appraiser judgment needed), list them explicitly when you hand back the files.

**If you skip Phase 5 for any reason:** You MUST tell the user "I have not completed the final review" before giving them the files.

**The orchestrator (main conversation) coordinates the agents but does NOT do their work.** The orchestrator:
- Launches agents via the Task tool
- Passes project path and template type
- Waits for agent results
- Presents review findings to the user
- Handles the user review pause between Phase 2 and Phase 3
- **Performs the Phase 5 final review directly before delivering output**

**When NOT to use the pipeline:**
- Quick single-cell fixes to an existing grid (do it directly)
- Answering questions about the data or documents (just read and answer)
- Updating data.md with user-provided values (do it directly)
- Investigating a specific comp (read the documents directly)

### Agent specs

All agent instructions live in `.claude/agents/`. All skill instructions live in `.claude/skills/`. Agents read these files for their detailed instructions. The orchestrator does not need to repeat the instructions — just pass the project path and template type, and reference the spec file.

## General Rules

- Assume the user is not technical. Use plain language. Don't assume they know Python, git, or the command line.
- Always give explicit instructions for how to do things in File Explorer / Finder.
- Always use the virtual environment for Python operations. Run scripts with `~/appraisal_ai/venv/bin/python` (full path). If the venv doesn't exist, create it with `python3 -m venv ~/appraisal_ai/venv`.
- Never install Python packages with `--break-system-packages`. Always use `~/appraisal_ai/venv/bin/pip install`.
- Never push to GitHub without asking.
- No monkey patches.
- **Never overwrite template grid values with `*** UPDATE ***` or empty data.** When filling the grid, only write a cell if comp_grid.md has real data for it. If a field is missing or marked `*** UPDATE ***`, leave the template's existing value. Same goes for adjustment rows — those are appraiser judgment; don't overwrite with defaults.
- **If it's not in the workfile, it doesn't exist.** Never assume facts that aren't in the project documents. If the narrative contains information (dollar amounts, contracts, LOIs, property history) that doesn't appear in any document in the project folder, it's a template artifact — flag it for removal. Don't ask the user to "confirm" template artifacts; just report that the information isn't supported by any source document.
- **Report truths, don't make appraiser judgments.** Adjustments (location, flood, conditions of sale, size), comp selection, and value conclusions are appraiser judgment. Report what the documents say — don't recommend adjustment amounts or suggest whether to include/exclude a comp. Flag factual issues (wrong price, wrong SF, non-arm's-length not disclosed) but leave the professional opinion to the appraiser.
- **Never use "N/A" as a field value in data.md.** If a field doesn't apply to this project (e.g., TE fields on a no-TE project, income fields on a land-only), set it to an empty string `''`. The build script inserts field values literally — "N/A" will appear as garbled text in the narrative (e.g., "N/A305N/A"). Empty strings cause the build script to skip the replacement entirely, leaving the template text for manual cleanup.

## Mandatory Pre-Delivery Review (Phase 5)

**This IS Phase 5 of the pipeline. Before handing ANY output file (draft, grid, or other deliverable) back to the user, you MUST complete this checklist. No exceptions. This applies whether you used the full `/run` pipeline, ran `/draft` or `/grid` individually, or made manual fixes.**

### For Excel Grid Files

1. **Inspect template structure first.** Before writing a single cell, dump the sheet names and identify which sheet to target. Never use `wb.active` — always open sheets by name (e.g., `wb['Original Land Sale Grid']`).
2. **Identify formula cells.** Read all cells in the target area and note which ones contain formulas. Never overwrite a formula cell.
3. **Identify appraiser-entered data.** Adjustment rows (location, flood, size, etc.) contain appraiser judgment. Never overwrite these unless the user explicitly asks.
4. **After writing, verify the output.** Re-open the saved file and dump the cells you changed. Confirm:
   - Correct sheet was written to
   - Formula cells are intact
   - Adjustment values were preserved
   - New values match data.md / comp_grid.md
5. **Check the other sheet(s).** If the workbook has multiple sheets, confirm you didn't accidentally modify the wrong one.

### For Word Narrative Files

1. **All replacements must use tracked changes.** Never do direct `<w:t>` text replacement. Every change must produce `<w:del>` + `<w:ins>` markup so the user can see and accept/reject.
2. **After writing, verify the output.** Extract text from the saved docx and spot-check:
   - Key field values (file number, address, dates, dollar amounts) are present and correct
   - No stale template values remain for fields that have project data
   - TE content was removed (if `has_te: false`)
   - Document opens without corruption (valid XML)
3. **Report what you couldn't fix.** List any reference values not found, any `*** UPDATE ***` fields remaining, and any stale values you spotted.

### Cross-Document Consistency

The land grid and the narrative are part of the same report. Data flows between them: the grid's comp data, adjusted PSFs, and subject info must match what appears in the narrative's comp summary table and land sales discussion. Before delivering either file:

1. **Grid ↔ Narrative comp summary table.** The comps in the narrative's "Comparable Sales After Adjustments" table must match the grid's final adjusted PSFs. Same comp numbers, same order, same values.
2. **Grid ↔ Narrative subject data.** Address, land SF, flood status, effective date, and other subject fields must be identical in both documents.
3. **Grid ↔ comp_grid.md.** Sale prices, dates, land SF, and other raw comp data in the grid must match comp_grid.md. If comp_grid.md has `*** VERIFY ***` or `*** UPDATE ***` for a field, leave the grid cell as-is.
4. **Don't guess at adjustments.** If you don't have confirmed data for a field (flood zone, location adjustment, etc.), leave it alone. The appraiser will fill it in.

### General

- **Never deliver without reviewing.** If you skip the review because of time or context, tell the user you haven't verified the output yet.
- **Compare output against data.md.** The final document must be consistent with the project data file.
- **Cross-check grid and narrative.** Every delivery must confirm the grid and narrative are telling the same story.
- **If something looks wrong, fix it before delivering.** Don't hand back a file with known issues and hope the user catches them.

## Lessons Learned

Running log of mistakes and the rules they produced. Read this on every session start. Update it when new lessons are learned.

### 2026-01-31 — Grid: Wrote to wrong Excel sheet
**What happened:** Used `wb.active` which defaulted to Sheet1 (sorted summary), not "Original Land Sale Grid." Overwrote existing data on the wrong sheet.
**Rule added:** NEVER use `wb.active`. Always open sheets by name. Inspect template structure before writing. (grid.md Step 3, CLAUDE.md Pre-Delivery Review)

### 2026-01-31 — Narrative: Direct text replacement without tracked changes
**What happened:** Applied 75 fixes as direct `<w:t>` text replacement. Changes were invisible to the user — no strikethrough/underline markup.
**Rule added:** ALL replacements must use tracked changes (`w:del` + `w:ins`). Never do direct text replacement. (CLAUDE.md Pre-Delivery Review, draft.md Critical Rules)

### 2026-01-31 — Grid: Overwrote appraiser adjustment values
**What happened:** Wrote default adjustment values (0) over existing appraiser-entered adjustments (e.g., -0.4 location, 0.05 flood).
**Rule added:** NEVER overwrite adjustment rows (29, 32, 35, 38). These are professional judgment. (grid.md Step 6, grid-builder.md, CLAUDE.md General Rules)

### 2026-01-31 — Delivered without reviewing output
**What happened:** Handed back files without verifying contents. User found wrong sheet, missing data, invisible changes.
**Rule added:** Mandatory Phase 5 Final Review before any delivery. Cannot hand files to user until review passes. (CLAUDE.md Phase 5, Pre-Delivery Review)

### 2026-01-31 — Didn't use agent team
**What happened:** Did all draft/grid work manually in main conversation instead of dispatching to the agent pipeline.
**Rule added:** When user asks to draft/build/run/review, MUST use the agent team via `/run`, `/draft`, `/grid`, or `/review` skills. (CLAUDE.md Mandatory Agent Workflow)

### 2026-01-31 — Comp writeups are narrative prose, not replaceable fields
**What happened:** The LAND SALE NO. sections contain full narrative descriptions of each comparable. These are template-specific prose that survives field-value replacement. The draft had the wrong comps described in detail.
**Rule added:** Created Comp Writer agent (`comp-writer.md`) that generates `comp_writeups.md` from comp_grid.md and source documents. Draft builder now reads this file and replaces the LAND SALE NO. sections with project-specific writeups. Falls back to template text (with warning) if comp_writeups.md doesn't exist. QA reviewer cross-checks narrative comp addresses against comp_grid.md.

### 2026-01-31 — TCE "No Interference" / "Restoration" clauses survived TE removal
**What happened:** The TE removal code looked for "Temporary Easement" and "temporary easement" but missed TCE covenant clauses using "No Interference", "Restoration", "upon the TCE."
**Rule added:** Added 6 new phrases to te_primary_phrases in draft.md: "No Interference", "upon the TCE", "Restoration", "obstruction will be placed, erected", "restore the TCE", "grantor covenants and agrees that no building."

### 2026-01-31 — Template artifacts in property history
**What happened:** A [template dollar amount] contract and [template dollar amount] LOI from the master template appeared in the new project's narrative. Not in any project document.
**Rule added:** "If it's not in the workfile, it doesn't exist." Added template artifacts to remove_sections in reference-data.md. (CLAUDE.md General Rules)

### 2026-01-31 — Smart quotes vs straight quotes in remove_sections
**What happened:** [template-specific text] was in remove_sections but survived removal. Word uses smart/curly quotes (Unicode right single quote U+2019) while the Markdown data file has straight apostrophes (U+0027). The string match failed.
**Rule needed:** Normalize smart quotes to straight quotes in both the XML text and the search strings before matching. (TODO: add to draft.md)

### 2026-02-01 — Skipped rebuild after auto-fix
**What happened:** Applied easement_grantor double-space fix to data.md but skipped the rebuild, leaving the draft with stale double-space text. The auto-fix loop protocol says to rebuild after applying fixes — no exceptions.
**Rule added:** After ANY auto_fix is applied to data.md, the rebuild is MANDATORY. Never skip it, even for "just one small fix." The rebuild catches cascading issues (other fields that depend on the fixed value, double-space normalization, etc.).

### 2026-02-01 — Word caps-styled fields are phantom reference values
**What happened:** `owner_name_caps` ("[template owner]") was listed in reference-data.md but doesn't exist as literal text in Word XML. Word stores the mixed-case text with `<w:caps/>` formatting — the all-caps rendering is visual only. The replacement always failed silently.
**Rule added:** Never list caps-styled variants as separate reference fields. The mixed-case replacement handles them automatically because Word's caps formatting applies to whatever the underlying text becomes. Removed `owner_name_caps` from reference-data.md.

### 2026-02-01 — Duplicate reference values cause first-wins collision
**What happened:** `hbu_as_vacant` and `hbu_as_improved` both referenced "Retail Commercial Building" in the template. Global find-replace changed all 3 instances to "Commercial Development" (the vacant value), leaving none for the improved replacement ("Continued interim use as a commercial property").
**Rule added:** When two fields share the same template text, add `_context` companion keys in reference-data.md (e.g., `hbu_as_vacant_context: "As Vacant"`). The build script detects these and uses context-aware replacement — finds the table row containing the context label and replaces only within that cell. Added to draft.md inline Python and "Known Replacement Edge Cases" documentation.

### 2026-02-01 — Split-run dollar formatting creates phantom reference values
**What happened:** `psf_concluded` ("$ 75") was listed in reference-data.md but the $ sign and number are stored in separate Word XML runs connected by spacing attributes, not literal space characters. The text never appears as a continuous string even after `merge_all_runs()`.
**Rule added:** Don't list spaced-dollar formats as reference values — they're phantom text. Use specific variants that exist as literal text (e.g., "$75 PSF", "$75.00 per square foot"). Removed `psf_concluded` from reference-data.md.

### 2026-02-01 — Comp writeup pages need page breaks and blank-line spacing
**What happened:** Generated comp writeups were "bunched up" — all detail lines ran together without spacing, and all 5 sales appeared on continuous pages instead of one sale per page with aerial on the facing page.
**Root cause:** `make_paragraph()` was cloning paragraph properties correctly (indent, font, alignment) but the code wasn't inserting: (1) blank paragraphs between each detail line, (2) page breaks between details and aerial, (3) page breaks between sales.
**Rule added:** Template layout is: sale details on page 1, page break, aerial placeholder on page 2 (heading + 20 empty lines for photo space), page break before next sale. Every detail field line must be followed by an empty paragraph. Headings are centered, detail lines are left-justified with hanging indent. Updated draft.md with `make_page_break_para()` helper and full layout documentation.

### 2026-02-01 — Large PDFs hang the Read tool
**What happened:** Agents tried to read large PDFs directly with the Read tool, which hung or errored out, stalling the pipeline.
**Rule added:** ALWAYS split PDFs before reading, regardless of file size. Use `split_pdf()` with `pages_per_chunk=3` for scanned docs (assessor cards, deeds), `pages_per_chunk=5` for text-heavy (CoStar, engagement letters). Updated subject-reader.md and comp-reader.md to make splitting mandatory, not optional.

### 2026-02-01 — CoStar buyer/seller = grantee/grantor
**What happened:** Comp writeups had `*** UPDATE ***` for grantor/grantee fields even though CoStar reports in the Comparables folder list buyer and seller for every transaction.
**Rule added:** CoStar "buyer" = grantee, "seller" = grantor. Always extract from CoStar, verify against deed when available. Added to comp-reader.md and comp-writer.md.

### 2026-02-01 — HBU context replacement used wrong labels
**What happened:** Context-aware replacement for HBU searched for "As Vacant"/"As Improved" labels in table rows, but the template uses paragraph format with "Before Condition"/"After Condition" labels. All 3 instances of "Retail Commercial Building" are actually hbu_as_improved.
**Rule added:** Replaced context system with direct HBU special case in draft.md — replace ALL instances of old HBU value with hbu_as_improved. hbu_as_vacant only appears in narrative discussion (appraiser writes it). Updated reference-data.md comments.

### 2026-01-31 — Made appraiser judgment calls
**What happened:** Suggested adjustment amounts and whether to include/exclude comps. These are professional opinions, not data extraction.
**Rule added:** "Report truths, don't make appraiser judgments." Flag factual issues but leave professional opinion to the appraiser. (CLAUDE.md General Rules)

### 2026-01-31 — Defaulted to CoStar for grantor/grantee instead of reading deeds first
**What happened:** Used CoStar buyer/seller as the primary source for grantor/grantee, then said "verify against deed." The appraiser expects us to read the deed first (authoritative source), cross-check against CoStar, and report discrepancies — saving them that manual step.
**Rule added:** Deed-first workflow: (1) Read deed, extract grantor/grantee. (2) Read CoStar buyer/seller. (3) Cross-check and report mismatches in notes. (4) Use deed values in output. Only fall back to CoStar if no deed available. Updated comp-reader.md and comp-writer.md.

---
> Source: [chadru/appraisal_ai](https://github.com/chadru/appraisal_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
