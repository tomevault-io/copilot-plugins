## task-extract-perform

> Processes PDF slide presentations by extracting deck content and end-of-deck activity instructions into paired markdown files, optionally emits a per-activity Python script when instructions call for runnable examples or practice, produces LaTeX write-ups per activity, aggregates all activities in a phase into one TeX document, compiles to PDF, zips the final PDF together with generated activity scripts, and treats that archive plus the full phase tree as the deliverable. Use when working with phased curriculum PDF decks, batch-processing presentations, consolidating phase activities, practical coding exercises alongside slides, or when the user mentions presentation-to-markdown pairs, optional activity Python helpers, phase-level TeX aggregation, or packaging a phase bundle.


# PDF phase presentations → markdown pairs → phase activity PDF

## Scope

- **Input**: One or more PDF presentations that belong to **phases** (phase identifiable from the file name).
- **Per presentation**: Two markdown files — **info** (deck context) and **activity** (instructions from the end of the deck).
- **Per presentation (optional)**: One **Python script** when instructions clearly benefit from a runnable example, simulation, data prep, or hands-on practice; omit when not applicable.
- **Per presentation**: One LaTeX file that is the **completed activity** (answer / deliverable).
- **Per phase**: One aggregated TeX file that includes all activity results for that phase, **compiled to PDF**, a **zip archive** containing that PDF and **all** `{base}_activity_script.py` files for the phase (see final workflow step), plus **all artifacts for that phase** including every markdown pair, each `_activity.tex`, and each generated script on disk (see layout below).

## PDF text vs scanned slides

- If the PDF has selectable text, extract with a text-based library (e.g. `pdfplumber` or PyMuPDF).
- If slides are image-only or OCR is needed, run OCR first (e.g. Tesseract + `pdf2image`, or any workflow that yields markdown or plain text per slide), then continue this pipeline using that extracted text as the source.

## Phase grouping (file names)

Infer **phase** from a consistent naming convention. Examples the user or project should adopt:

- `phase1_01_intro.pdf`, `phase1_02_models.pdf` → phase `phase1`
- `P2-SlideDeck-TopicA.pdf` → phase `P2`

If names are ambiguous, ask the user once for the rule (regex or prefix/split pattern), then apply it to all files in the batch.

## Output layout (recommended)

Use a dedicated folder per phase so aggregation is obvious:

```text
output/
  phase1/
    decks/
      01_intro_info.md
      01_intro_activity.md
      01_intro_activity.tex         # completed activity (LaTeX fragment)
      01_intro_activity_script.py   # optional: only if applicable (see below)
    phase1_activities.tex           # \input{} or \include{} of each *_activity.tex
    phase1_activities.pdf           # compiled output
    phase1_deliverable.zip          # PDF + scripts/… (see workflow step 12)
```

Adjust names to match the user’s naming scheme; keep the **pairing rule** fixed: one base name → `*_info.md`, `*_activity.md`, `*_activity.tex`. **Optional script** name: `{base}_activity_script.py` (same `{base}` as the markdown pair, e.g. `01_intro_activity_script.py`).

**Phase deliverable:** Treat the whole `output/<phase>/` tree (PDF, master `.tex`, `decks/*.md`, `decks/*_activity.tex`, **any** `decks/*_activity_script.py`, and **`{phase}_deliverable.zip`**) as the consolidated output for the phase. The zip is the handoff bundle for **compiled PDF + scripts**; other files remain for editing and rebuilds.

## Workflow

Follow these steps in order.

1. **Read the presentation** (PDF). Determine phase from the file name.
2. **Extract relevant information** into an `.md` file (concepts, definitions, examples from the deck — not the activity block yet). This is the **info** markdown.
3. **Read the activity instructions** at the end of the presentation (last slides or labeled section).
4. **Extract the instructions** into a **second** `.md` file (activity-only, verbatim or clearly structured).
5. **Each presentation** must yield a **pair** of markdown files: **info** + **activity instructions**.
6. **Optional Python script for this activity** (after the activity `.md` exists):
   - Decide whether a **runnable Python script** would materially improve results: e.g. numerical experiment, simulation, data/file processing, plotting, algorithm demo, scaffold for exercises that ask for code, or “try it yourself” practicals aligned with the instructions.
   - **Clearly applicable:** Write `{base}_activity_script.py` next to the other deck artifacts. Keep it focused on the activity (docstring referencing the activity instructions, minimal deps, `if __name__ == "__main__":` entry when useful). Use only standard library unless the user or project already specifies dependencies.
   - **Clearly not applicable** (pure reflection/essay, untyped discussion, multiple-choice-only, no computational or practical hook): **do not** create a script.
   - **Unclear:** **Ask the user** whether they want a script for this activity and what it should demonstrate or automate; only create it after they confirm (or decline).
7. **Create a final `.tex` file** with the **result of the activity** (the completed work: answers, reflection, design, etc.), using the activity `.md` as the spec for what to deliver. If a companion `{base}_activity_script.py` exists, **reference it** in the write-up (filename and one-line purpose); see [reference.md](reference.md).
8. **Repeat** steps 1–7 for **every presentation in the same phase** (same inferred phase id).
9. **Aggregate** all per-presentation activity `.tex` files for that phase into a **single master `.tex`** (see [reference.md](reference.md) for a minimal skeleton).
10. **Compile** the aggregated `.tex` to PDF (e.g. `latexmk -pdf phase1_activities.tex` or `pdflatex` twice if needed for references/TOC).
11. **Hand off the phase output:** PDF + master TeX + all `decks/` markdown, LaTeX fragments, and **every `{base}_activity_script.py` generated during this phase** (same folder tree).
12. **Create a zip archive** of the **final phase PDF** and **all** `{base}_activity_script.py` files under that phase directory. Default output: `{phase}_deliverable.zip` next to the PDF (e.g. `output/phase1/phase1_deliverable.zip`). Archive layout: PDF at the **root** of the zip; scripts under **`scripts/`** with paths relative to the phase folder (e.g. `scripts/decks/01_intro_activity_script.py`). If no scripts were generated for that phase, the zip still contains the PDF only. Run [scripts/zip_phase_deliverable.py](scripts/zip_phase_deliverable.py) or reproduce the same layout by hand.

## Quality checks

- [ ] Info `.md` reflects deck teaching content; activity `.md` matches end-of-deck instructions.
- [ ] Every deck in the phase has both `.md` files and one completed `_activity.tex`.
- [ ] Optional scripts: created **only** when clearly useful or after user confirmation when unclear; **no** orphan scripts for purely non-coding activities.
- [ ] Each `{base}_activity_script.py` is import-safe or clearly entrypoint-only, matches the activity wording, and is included in the phase deliverable folder.
- [ ] Master TeX includes every activity in phase order (sort by inferred sequence from file names if present).
- [ ] PDF builds without errors; fix missing packages or Unicode issues in LaTeX.
- [ ] `{phase}_deliverable.zip` exists, opens cleanly, and contains the compiled `*_activities.pdf` plus every `*_activity_script.py` for that phase (under `scripts/`).

## Utility scripts

Install Python dependencies once:

```bash
pip install -r scripts/requirements.txt
```

Run all script paths from the skill root `pdf-phase-presentation-workflow/`, or `cd scripts` and invoke with `python ./scriptname.py` (sibling imports resolve via the script directory).

| Script | Purpose |
|--------|---------|
| [scripts/extract_presentation_pair.py](scripts/extract_presentation_pair.py) | One PDF → `{base}_info.md` + `{base}_activity.md` (last *N* pages = activity; tune `--activity-pages`). |
| [scripts/group_pdfs_by_phase.py](scripts/group_pdfs_by_phase.py) | List PDFs grouped by phase (`--regex` with `(?P<phase>...)` or `--split-delimiter` + `--split-index`). Add `--json` for machine-readable output. |
| [scripts/batch_extract_for_phases.py](scripts/batch_extract_for_phases.py) | Batch: group all PDFs in a directory and extract pairs into `output/<phase>/decks/`. |
| [scripts/new_activity_tex_stub.py](scripts/new_activity_tex_stub.py) | Write one minimal `\\section{...}` LaTeX **fragment** (no preamble) for hand-filling or agent completion. |
| [scripts/batch_activity_stubs.py](scripts/batch_activity_stubs.py) | For each `*_activity.md` in a `decks/` folder, write matching `*_activity.tex` stub if missing. |
| [scripts/build_phase_master_tex.py](scripts/build_phase_master_tex.py) | Generate master `.tex` with `\\input{}` for every matching `*_activity.tex` (sorted). |
| [scripts/compile_phase_pdf.py](scripts/compile_phase_pdf.py) | Run `latexmk -pdf` (fallback: `pdflatex` ×2). Requires a LaTeX distribution on PATH. |
| [scripts/zip_phase_deliverable.py](scripts/zip_phase_deliverable.py) | Write `{phase}_deliverable.zip`: root = `*_activities.pdf`; `scripts/` = all `*_activity_script.py` under the phase dir. Optional `--pdf`, `--out`, `--require-scripts`. |

**Example (regex phase prefix):**

```bash
python scripts/batch_extract_for_phases.py /path/to/pdfs --regex '^(?P<phase>phase[0-9]+)_' --out ./output --activity-pages 4
python scripts/batch_activity_stubs.py ./output/phase1/decks
# Edit ./output/phase1/decks/*_activity.tex with completed work (or let the agent fill from *_activity.md).
python scripts/build_phase_master_tex.py \
  --master ./output/phase1/phase1_activities.tex \
  --search-dir ./output/phase1 \
  --glob '**/*_activity.tex' \
  --title 'Phase 1 activities' \
  --author 'Course' \
  --exclude-substring '_activities.tex'
python scripts/compile_phase_pdf.py ./output/phase1/phase1_activities.tex
python scripts/zip_phase_deliverable.py --phase-dir ./output/phase1
```

**Agent vs automation:** Steps 1–5 (paired markdown), 9–10 (aggregate + compile), and **12 (zip)** map directly to these tools. Step 7 (real activity answers in LaTeX) and **step 6 (optional `*_activity_script.py`)** are substantive: the agent judges fit, asks when unsure, implements scripts and prose. Pipeline scripts do **not** auto-generate activity Python; that remains agent-led per the rules above.

## Additional resources

- Master TeX aggregation pattern: [reference.md](reference.md)

---
> Source: [carlosValerio5/task-extract-perform](https://github.com/carlosValerio5/task-extract-perform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
