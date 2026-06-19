## paper-shred

> |


# /paper-shred — PDF → structured folder of markdown

Shred a scientific PDF into individually addressable components, following the
"biomedical literature as a filesystem" principle (https://gxl.ai/blog/biomedical-literature-as-a-filesystem).
Each paper becomes a folder where sections, figures, tables, and references are
separate files an agent can grep/read directly without ingesting the whole doc.

**Two extractors run in parallel:**
- **marker-pdf** — primary source for body text. Best at preserving `<sup>` citation wrappers, bold/italic emphasis, and scientific notation (e.g. `Atg7<sup>flox/flox</sup>`).
- **docling** — auxiliary source for document-level metadata. Best at detecting the document title, providing a clean heading scaffold (uniform H2s instead of marker's inconsistent levels), and counting figures/tables.

Claude reconciles them: body content from marker, title + structural scaffold from docling.

**Argument**: `$ARGUMENTS` should be a path to a PDF. If empty or invalid,
tell the user: `Usage: /paper-shred /path/to/document.pdf [output_parent_dir]`.

If `$ARGUMENTS` is a **directory**, switch to batch mode (see below) instead of
running the per-PDF Claude flow — for >5 PDFs the per-paper restructuring is
impractical in a single session.

---

## Batch mode (directories of >5 PDFs)

```
python ${CLAUDE_SKILL_DIR}/bin/shred_batch.py <directory> [--require-caption]
```

Walks the directory for `*.pdf` (skipping symlinks and anything under `_raw/`),
runs the same extract → clean → audit → split → write → post-pass pipeline per
file using deterministic heuristics — no Claude in the loop. Idempotent: a
folder with `meta.json + README.md` is skipped on re-run; the extract.sh cache
makes "delete user-facing output, re-shred" finish in seconds. Per-paper status
is appended to `<directory>/_shred_log.jsonl`. Expect 5–10 min/PDF on CPU
(50 PDFs ≈ 5–8 hours wall time). Use `--require-caption` on Cell/Nature
collections to drop figure panels without an anchored caption.

The single-PDF Claude-assisted flow below stays the right choice when you want
careful structural decisions (grants, theses, reviews with unusual layouts).

---

## Step 1 — Validate and resolve paths

Parse `$ARGUMENTS`:
- First token: PDF path (required, must end in `.pdf` and exist).
- Second token (optional): parent directory for the output folder. Default:
  the directory containing the PDF.

Compute:
- `STEM` = PDF basename without `.pdf`, sanitized (replace whitespace/punct with `_`)
- `OUT_DIR` = `<output_parent_dir>/<STEM>`
- `WORK` = `<OUT_DIR>/_raw` (working dir for marker output)

If `OUT_DIR` already exists with a `README.md`, ask the user whether to
overwrite, append a suffix (`_v2`), or abort. Don't silently overwrite.

---

## Step 2 — Run extractors (marker-pdf + docling, parallel)

```bash
bash ${CLAUDE_SKILL_DIR}/bin/extract.sh "<PDF>" "<WORK>"
```

This launches both extractors in background and waits for both. Stdout emits
`key=value` lines: `marker_md`, `marker_meta`, `marker_dir`, `docling_md`,
`docling_meta`, `docling_pics`, `docling_title`, `stem`, `marker_venv`,
`docling_venv`. Capture all of them.

If marker fails the script aborts. If docling fails (or its venv is missing),
the script emits a warning and continues with marker-only output — `docling_md`
and friends will be empty strings; downstream logic must handle that.

**Runtime expectations** (CPU, 5-page document):
- First run: 5–10 minutes (marker downloads ~2 GB of surya models; docling
  downloads ~50 MB of RapidOCR + layout models).
- Cached: marker ~1–7 minutes depending on page count, docling ~1–3 minutes,
  running in parallel — wall time dominated by marker.
- A GPU drops marker to ~0.2 s/page; not required.

---

## Step 3 — Mechanical pre-pass

```bash
"$venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/mechanical_clean.py \
    "$marker_md" "$WORK/cleaned.md"
```

This strips Zotero/google-doc tracking URLs, drops `{N}-----` page-break
separators, replaces `<br>` tags, and normalizes whitespace. It does **not**
attempt heading promotion or reference splitting — that's your job in Step 4.

Note the stderr summary line: chars, image count, table count, and citation
counts broken down by style (`sup`, `bracket`, `authoryear`) plus the dominant
style. Use the dominant-style count to populate `meta.json.n_citations_inline`
and record the style in `meta.json.citation_style`. The author-year counter is
loose by design (it also matches bibliography entries) — when the dominant
style is `authoryear` the count is roughly "inline citations + bibliography
entries", so subtract the reference-list size if you need a tight inline count.

---

## Step 4 — Read and plan the structure

Read `$WORK/cleaned.md`. Then plan the folder layout:

### 4a. Identify document type

From cues in the text, classify as one of: `paper`, `grant`, `review`,
`preprint`, `thesis`, `protocol`, `other`. Heuristics:
- `paper`: Abstract + Introduction + Results + Discussion + Methods/Materials
- `grant`: Specific Aims, Vision/Approach, Significance, Innovation, Objectives,
  Research Strategy, Budget, Biosketch, Letters of Support
- `review`: lots of citations, no Methods/Results, narrative section names
- `preprint`: paper-shaped, with bioRxiv/medRxiv/arXiv markers
- `thesis`: Chapters 1..N, Acknowledgements, Declaration

### 4b. Extract bibliographic metadata

Pull from the document text (do not invent):
- `title`: **run the title picker first**:

  ```bash
  "$marker_venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/pick_title.py \
      "$docling_meta" "$WORK/cleaned.md" "<source_pdf_path>"
  ```

  This script tries an explicit body `Project title` field first (high-confidence
  signal that beats docling on form-shaped documents), falls back to the docling
  title if it passes rejection rules (not a Section/Annex prefix, length 15–200,
  not an all-caps banner), and last-resorts to the filename stem. Output is a
  JSON object with `recommended`, `source` (`body_table | body_heading | docling
  | filename`), `rejected_docling`, `rejection_reason`, and `warnings`. Use the
  `recommended` value as the title and copy any `warnings` into `meta.json.notes`.
  If `source == "filename"`, surface that prominently in the red-flags report.
- `authors`: usually below title, may be comma- or superscript-separated. If the
  document is first-person without a visible author block (common for grant
  Vision/Approach sections), set `authors: null` and flag in `notes` — do not
  infer from session context unless the user explicitly confirms.
- `year`: from header/footer or first-page metadata
- `doi`: regex `10\.\d{4,9}/[^\s]+` if present
- `journal` / `funder` / `grant_id` / `agency` if obvious

If a field is genuinely absent, set it to `null`. Do **not** hallucinate.

### 4c. Decide section split (cap at ~8 top-level sections)

For each top-level section, you'll create one `sections/NN_<slug>.md` file.
**Cap at ~8 files.** If the document has deeper structure (e.g. 12 H2 headings),
group related ones under a single section file and keep the sub-structure as
H2/H3 *inside* that file. The README's TOC reflects the file-level split, not
every H2.

**Use the docling heading list as a sanity check.** `docling_meta.json` contains
a `headings: [{level, text}, ...]` list. Compare it to the section split you
plan from the marker body — if docling sees 5 top-level headings and you plan
3 sections, you're probably under-splitting (or vice-versa). Trust your reading
of the marker body for the actual content but use docling to catch missed
boundaries.

Section slug rules:
- lowercase, snake_case, ≤ 25 chars
- numeric prefix `01_`..`NN_` reflecting reading order
- title-bearing: prefer `02_introduction.md` over `02_intro.md`; for grants use
  `03_objective_1.md` not `03_obj1.md`

Examples:
- Nature paper → `01_abstract.md`, `02_introduction.md`, `03_results.md`,
  `04_discussion.md`, `05_methods.md`
- UKRI FLF grant → `01_vision.md`, `02_approach.md`, `03_objective_1.md`,
  `04_objective_2.md`, `05_objective_3.md`, `06_long_term_plan.md`
- NIH R01 → `01_specific_aims.md`, `02_significance.md`, `03_innovation.md`,
  `04_approach.md`, `05_aim_1.md`, `06_aim_2.md`, `07_aim_3.md`

**Three rules learned from batch-shredding 50 lab papers:**

1. **Drop the title-only first section.** PMC-archived papers commonly have
   `# Title` and `# Abstract` as sibling H1 headings, with only authors and
   affiliations between them. After section-splitting, if the first section's
   title matches the chosen `meta.json.title` (slug-normalised) AND its body
   is < 800 chars, drop it and renumber. Without this, section 01 is just an
   author block.

2. **H3-fallback anchors.** Cell Press papers (MolCell, CellReports) often
   come out of marker as all-H3 with no H1 or H2. If neither H1s nor H2s have
   ≥ 3 entries after banner filtering, fall back to H3 anchors before
   defaulting to a single-section dump. Without this, an entire MolCell paper
   collapses to one section.

3. **Filename metadata convention.** If the source PDF filename matches
   `YYYY-MM_Journal_topic_(PMID\d+|DOI[\d.]+)\.pdf`, parse `year`, `journal`,
   and the identifier from it — these populate `meta.json` for free when
   docling/marker can't find them in the body. Optional but cheap to support;
   the Ule lab convention exposed how often it Just Works.

### 4d. Inventory figures and tables

**Run the figure filter first** to separate scientific figures from page
banners and logos. Both extractors emit images for every visual element they
detect — including page-running headers, journal logos, and partner-logo
strips in grant docs. Trusting their raw counts gives a wildly inflated
`n_figures`.

```bash
"$marker_venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/filter_figures.py "$WORK"
```

This writes `<WORK>/figure_audit.json` listing each image with width, height,
area, aspect ratio, and a `kept` flag. Default thresholds drop images where
`min_dim < 200`, `aspect > 5`, or `area < 50,000` — these catch the common
patterns (banner strips ~822x90, logo thumbnails ~25x29). The script does NOT
delete files; you remain free to override its decision for borderline cases
(e.g. small but legitimate diagrams).

Then for each remaining image planned for `figures/`:
- figures: `figures/figure_01.jpeg` (and a sidecar `figures/figure_01.md`)

For tables, **run the table classifier** to separate data tables from
admin form-fields and from prose blocks marker wrongly piped:

```bash
"$marker_venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/classify_tables.py \
    "$WORK/cleaned.md" "$WORK/table_audit.json"
```

Each block is classified `data | admin | prose | empty`. Only `data` tables
go to `tables/table_NN.md` (and `.csv` when rectangular). `admin` tables stay
inline in their owning section file. `prose` blocks are 1-column pipe-wrapped
text that should be unwrapped back to plain prose in the section file (most
common case: marker wrapping a references list in pipes).

The classifier is opinionated but coarse — if a borderline `data` verdict on
a small table is actually admin (e.g. a 4-row co-applicant list), override
manually and note it in the red-flags report.

**Cross-check with docling.** `docling_meta.n_pictures` and the files in
`docling_pics/` show what docling detected — but apply the same filter. If
the audit kept docling extras that marker didn't extract, those may be real
diagrams marker missed; copy them into `figures/`. Note any decision in the
red-flags report.

---

## Step 5 — Restructuring rules (critical — read carefully)

When you split content into section files, you are doing **structural
rewriting only**. Hard constraints:

1. **Do not modify prose.** Sentence text is sacrosanct. No paraphrasing,
   no "fixing" awkward phrasing, no expanding abbreviations.
2. **Do not add or remove citations.** Every `<sup>N</sup>` in the input must
   appear in exactly one section file. If you spot a likely lost citation
   (e.g. `dataset. 13 Reporters`), wrap it as `<sup>13</sup>` and note this
   in the final red-flags report.
3. **Preserve all figures, tables, equations, and inline formatting** exactly
   as written.
4. **Headings within a section file** start at H2 (since the file has its own
   H1 title). Reflect the document's internal sub-structure — don't flatten.
5. **Reference list goes to `references.md` only**, never duplicated in a
   section.
6. **If a paragraph straddles two sections** (rare — happens when marker
   misplaces a heading), assign it to the section whose topic it discusses.

---

## Step 6 — Write the folder

Create `OUT_DIR/` with this layout:

```
<OUT_DIR>/
├── README.md
├── meta.json
├── sections/
│   ├── 01_<slug>.md
│   └── ...
├── figures/
│   ├── figure_01.jpeg            (copy from _raw/marker_out/<stem>/_page_*_Figure_*.jpeg)
│   └── figure_01.md              (caption + AI description)
├── tables/
│   └── table_01.md (and .csv when applicable)
├── references.md
└── _raw/
    ├── source.pdf                (symlink to original PDF)
    ├── marker.md                 (raw marker output, renamed)
    ├── marker_meta.json
    └── cleaned.md
```

### 6a. README.md template

```markdown
---
title: "<title>"
authors:
  - "<author 1>"
  - "<author 2>"
year: <year>
type: <paper|grant|review|preprint|thesis|protocol|other>
doi: <doi or null>
journal: <journal or null>
source: _raw/source.pdf
extracted: <YYYY-MM-DD>
extractor: marker-pdf + paper-shred
tags:
  - <type>          # e.g. paper, grant
---

# <title>

> **Source:** `<original_pdf_filename>`
> **Type:** <type> · **Year:** <year> · **DOI:** <doi or "—">

## Abstract / Summary

<verbatim abstract from the document, or the equivalent opening summary>

## Sections

- [[sections/01_<slug>|<Section 1 title>]]
- [[sections/02_<slug>|<Section 2 title>]]
- ...

## Figures

- [[figures/figure_01|Figure 1 — <caption first sentence>]]
- ...

## Tables

- [[tables/table_01|Table 1 — <caption first sentence>]]
- ...

## References

[[references|<N> references]]
```

Keep tags minimal: just the document `type`. Topic tags require user context
that this skill doesn't have — leave them for the user to add.

### 6b. Section file template

```markdown
# <Section title>

<verbatim content from cleaned.md, with H2/H3 sub-headings as appropriate>
```

### 6c. Figure sidecar template

```markdown
# Figure <N>

![Figure <N>](figure_<NN>.jpeg)

**Caption (verbatim):** <caption text from the document>

**Description:** <1–2 sentences summarizing what the figure shows, derived
from the caption and surrounding context. If no caption is detectable, write
"No caption detected; derived from context: ..." and describe what you can
infer.>
```

### 6d. references.md template

```markdown
# References

1. <full reference 1>
2. <full reference 2>
...
```

One reference per line for greppability. Parse from whatever marker emitted
(numbered, author-year, etc.) — common section headers to look for:
`References`, `Bibliography`, `Literature Cited`, `Works Cited`, `REFERENCES`.

**Run `extract_refs.py` first** to find the section or fall back to scanning
the body for PMID/DOI tokens:

```bash
"$marker_venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/extract_refs.py \
    "$WORK/cleaned.md"
```

Output JSON has:
- `has_references_section`: if true, splice that block into `references.md`.
  Use `section_start_line` to locate it.
- `entries`: list of `{id, snippet}` for every PMID/DOI found in the body.
  When the source has no formal reference section (common for grant EoIs and
  forms with author-year + PMID inline citations), use these as `references.md`
  entries — one per line — and note in `meta.json.notes` that the list was
  reconstructed from inline identifiers, not parsed from a bibliography.

If both `has_references_section` is false AND `entries` is empty, the source
has no machine-parseable references at all — most often because they were
attached as a separate PDF (Longitude Prize). State this explicitly in
`references.md` and link the side-attachment.

### 6e. meta.json template

```json
{
  "title": "...",
  "authors": ["..."],
  "year": 2026,
  "type": "grant",
  "doi": null,
  "journal": null,
  "funder": null,
  "grant_id": null,
  "source_pdf": "_raw/source.pdf",
  "extracted_at": "<ISO 8601 timestamp>",
  "extractor": "marker-pdf + paper-shred 0.1.0",
  "n_pages": <int or null>,
  "n_figures": <int>,
  "n_tables": <int>,
  "n_references": <int>,
  "citation_style": "sup | bracketed | authoryear | null",
  "n_citations_inline": <count of dominant-style citations in body>,
  "sections": ["01_...md", "02_...md", ...]
}
```

### 6f. _raw/ artifacts

- Symlink: `ln -sf "<absolute_pdf_path>" "<OUT_DIR>/_raw/source.pdf"`
- Move/copy: `marker_dir/<stem>.md` → `_raw/marker.md`
- Move/copy: `marker_dir/<stem>_meta.json` → `_raw/marker_meta.json`
- Keep: `_raw/cleaned.md` already there from Step 3
- Keep: `_raw/docling_out/{docling.md, docling_meta.json, docling_pictures/}`
  already there from Step 2 (when docling ran)

Use `cp -r` not `mv` if you want to preserve the marker_out tree for re-runs.

---

## Step 6.5 — Run `clean_sections.py` post-pass

After all section files are written but before verification, run the
post-pass cleaner. It strips marker artefacts that survived the pre-pass
because they needed Claude's section-split decisions first to know what
counts as cruft (vs. legitimate content):

```bash
"$marker_venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/clean_sections.py "<OUT_DIR>"
```

Removes:
- Embedded `#### Figure N. ...` heading + caption blocks (already in `figures/figure_N.md`)
- Paragraph-form `Figure N. ...` captions
- Page-banner H1 headings (`# Article`, `# Authors`, `# In brief`, `# Correspondence`, `# Highlights`, `# Graphical abstract`, `# SUMMARY`)
- Author affiliation footnote lines (`<sup>N</sup>Department of …`)
- Joins paragraphs split mid-sentence by removed-cruft page boundaries

Idempotent — safe to run multiple times. Logs per-file deltas to stderr.

## Step 7 — Verification and report

Run a quick sanity pass and report to the user:

```bash
python3 - <<'PY'
import pathlib, json, re
out = pathlib.Path("<OUT_DIR>")
meta = json.loads((out / "meta.json").read_text())
sections = list((out / "sections").glob("*.md"))
figures = list((out / "figures").glob("*.jpeg"))
tables_md = list((out / "tables").glob("*.md"))
refs_md = (out / "references.md").read_text() if (out / "references.md").exists() else ""
n_refs_listed = sum(1 for l in refs_md.splitlines() if re.match(r"^\d+\.\s", l))
all_text = "\n".join(p.read_text() for p in sections)
n_sup = all_text.count("<sup>")
print(f"sections: {len(sections)}")
print(f"figures:  {len(figures)}")
print(f"tables:   {len(tables_md)}")
print(f"refs:     {n_refs_listed} (meta: {meta.get('n_references')})")
print(f"<sup>:    {n_sup} (meta: {meta.get('n_citations_inline')})")
PY
```

Report to the user:
1. The output folder path
2. Section count, figure count, table count, reference count
3. **Red flags to inspect manually:**
   - Section count > 8 → grouping rule violated
   - `<sup>` count in sections ≠ count in cleaned.md (excluding refs-section
     entry markers) → citations got dropped during the section split
   - Reference count < 5 in a paper or grant → reference parser missed the section
   - Any section file < 200 chars → likely a heading-detection misfire
   - Title or authors null when document clearly has them
   - Marker and docling disagree on figure count by >1 → check docling's
     extras under `_raw/docling_out/docling_pictures/` for legitimate
     diagrams marker missed
   - Marker and docling disagree on heading count by >2 → re-read your
     section split decision
4. The `meta.json` summary

End with: `Folder ready: <OUT_DIR>`. Don't volunteer next steps unless asked.

---

## Notes on adaptation

- For **non-English documents**, the section-name heuristics in Step 4a still
  work because they're based on document shape, not specific words. The
  resulting section slugs should be in English (translate the section name)
  to keep the convention consistent across the vault.
- For **scanned PDFs** (no text layer), `extract.sh` handles OCR via
  marker/surya automatically, but expect more lost citations and table noise.
- For **very long documents** (> 50 pages, e.g. theses), Step 4c's 8-section
  cap may compress chapters into one file — ask the user whether to relax the
  cap to "one file per chapter" before proceeding.

---
> Source: [MathieuBo/paper-shred](https://github.com/MathieuBo/paper-shred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
