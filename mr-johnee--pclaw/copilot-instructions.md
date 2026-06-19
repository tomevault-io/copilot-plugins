## pclaw

> Systematic literature-mining pipeline for discovering novel covalent reactive handles capable of forming stable bonds with specified biological side-chain functionalities (including cysteine, lysine, serine, histidine, tyrosine, threonine, aspartate, glutamate, and methionine) under intended probe-use conditions. Use when a user asks to survey recent chemistry literature for novel covalent reaction partners targeting a specific residue or functional group, search for side-chain-reactive covalent motifs, design new reactive handles for chemical probes or covalent ligands, or produce an academic report of novel candidates with validated SMILES, rendered structures, and primary-literature citations.


# Covalent Probe Discovery

A five-step pipeline that turns a target residue or side-chain functionality
into a short, paper-style Markdown report of novel covalent reactive handles,
each accompanied by a validated SMILES, an RDKit-rendered structure, and
citations to the primary literature.

The pipeline is designed to be reproducible and auditable: every artifact
produced by a run is retained, every candidate in the final report is backed
by a locator in the source PDF, and the known-warhead exclusion list is
versioned alongside the skill itself.

## Workflow Overview

```
Step 1  Generate keywords (keyword-generation prompt)   keywords.txt
Step 2  PubMed search                                   pubmed_results.json
Step 3  Relevance scoring (relevance-scoring prompt)    shortlist.json
Step 4  Attempt OA PDF acquisition                      pdfs/, download_log.json
Step 5  Extract, validate, render, compose report       candidates.enriched.json,
                                                        images/*.png, report.md
```

Steps 1, 3, 5a, and 5c are LLM-driven and follow the prompts in
[references/](references/). Steps 2, 4, and 5b are deterministic scripts under
[scripts/](scripts/).

## Artifacts produced per run

The run directory is chosen by the caller (not the skill). Inside it the
pipeline produces:

| File | Produced by | Audience |
|------|-------------|----------|
| `keywords.txt`                | Step 1 | reproducibility |
| `pubmed_results.json`         | Step 2 | audit |
| `shortlist.json`              | Step 3 | audit |
| `pdfs/<doi>/paper.pdf`        | Step 4 | evidence source when available |
| `pdfs/download_log.json`      | Step 4 | acquisition audit |
| `candidates.json`             | Step 5a | structure-extraction prompt output |
| `candidates.enriched.json`    | Step 5b | validated candidate record |
| `images/candidate_NNN.png`    | Step 5b | figures for the report |
| `candidate_validation.json`    | Step 5b audit | JSON Schema / evidence audit |
| `report.md`                   | Step 5c | **the deliverable** |

`report.md` reads like a short paper. Confidence scores, evidence quotes,
per-claim locators, structure-source locators, and manual-review status live
in `candidates.enriched.json`; they do not appear as database fields in the
human report.

## Setup

Install this repository as a skill folder first (for example under
`~/.openclaw-autoclaw/skills/pClaw/` for OpenClaw / AutoClaw). The installed
folder must contain this `SKILL.md`, plus the bundled `scripts/`,
`references/`, and `data/` directories. The folder name may be `pClaw`; the
registered skill name is the `name` field at the top of this file:
`covalent-probe-discovery`.

### Python dependencies

The pipeline has three dependency tiers:

- Step 2 (`scripts/pubmed_search.py`) uses only the Python standard library.
- Step 4 (`scripts/download_pdf.py`) requires `httpx` and network access to
  Europe PMC, PMC's OA service, and the Unpaywall API.
- Step 5b (`scripts/smiles_to_image.py`) requires `rdkit`.

Install with pip:

```bash
python3 -m pip install -r requirements.txt
```

If `pip` cannot install `rdkit` on your platform (common on Apple Silicon
outside conda), use a conda-compatible environment:

```bash
conda create -n covalent-probe python=3.12 -c conda-forge rdkit httpx
conda activate covalent-probe
```

### Required configuration

The Unpaywall API requires a contact email. Set it before running Step 4:

```bash
export CHEM_PDF_UNPAYWALL_EMAIL="you@example.com"
```

Keep real local values in your shell profile or a private `.env` file. Do not
commit real emails, API keys, or tokens. See [env.example.sh](env.example.sh)
for the full list of optional environment variables. The skill does not
auto-load `.env` files.

### Non-package prerequisites

- Network access to Europe PMC, PMC's OA service, and the Unpaywall API.
- The bundled [data/known_warheads.json](data/known_warheads.json) exclusion
  list (shipped with this skill; override with `COVALENT_PROBE_KNOWN_WARHEADS`
  or `--known-warheads` on `run_pipeline.py` if you maintain a local copy).

No extra command-line tools are required — the pipeline does not call
`pdftotext`, `poppler`, `ghostscript`, `openbabel`, `java`, or `node`.

## Running the pipeline

All examples below assume the current working directory is the run directory
and `$SKILL_DIR` points to the skill root:

```bash
export SKILL_DIR="/absolute/path/to/pClaw"
```

An orchestrator for the deterministic steps (2, 4, 5b) is provided:

```bash
python3 "$SKILL_DIR/scripts/run_pipeline.py" --keywords keywords.txt
```

See `python3 "$SKILL_DIR/scripts/run_pipeline.py" --help` for options. The
sections below document each step individually.

## Step 0 — Confirm survey scope with the caller

Before generating keywords, ask the caller two scope questions and record the
answers verbatim in the run header of the final report:

1. **Publication-year window.** "Should I restrict the survey to a publication
   year range (e.g. 2015–2025)?" For reproducible academic surveys a hard
   window is strongly recommended — pick an inclusive `[min_year, max_year]`
   pair the caller can quote in their Methods section. Default if the caller
   declines: no year restriction.
2. **Journal whitelist (optional).** "Should I restrict the search to specific
   journals by abbreviation (e.g. *J. Am. Chem. Soc.*, *Angew. Chem. Int.
   Ed.*, *Nat. Chem. Biol.*)?" Default: no whitelist. Warn the caller that a
   journal whitelist hides good chemistry in less obvious venues, so it
   should be used only when the survey is *explicitly* scoped to a venue
   subset.

Pass both answers through to Step 2 via the `--min-year` / `--max-year` /
`--journal` flags so the filters apply at the PubMed query layer rather than
post-hoc, and so they appear in `pubmed_results.json`'s `filters` block for
audit.

## Step 1 — Generate search keywords

Read [references/prompt-keyword-generation.md](references/prompt-keyword-generation.md).
Produce 24–36 keywords with five-layer coverage and write them to
`keywords.txt` (one per line). Keywords are chemistry-led and do **not**
encode year or journal — those are applied at the search layer in Step 2.

## Step 2 — PubMed search

```bash
python3 "$SKILL_DIR/scripts/pubmed_search.py" \
    --query-file keywords.txt \
    --retmax 20 \
    --sort date \
    --max-pages 50 \
    --min-year 2015 --max-year 2025 \
    --pretty > pubmed_results.json
```

Add a journal whitelist only if the caller asked for one in Step 0:

```bash
    --journal "J Am Chem Soc" \
    --journal "Angew Chem Int Ed Engl" \
    --journal "Nat Chem Biol"
```

Behaviour:

- Reviews, systematic reviews, and meta-analyses are excluded at the PubMed
  query layer by default. Pass `--include-reviews` to override.
- `--min-year` / `--max-year` apply an inclusive PubMed `[PDAT]` window. Both
  are optional; omit a bound to leave that side open. Strongly recommended
  for reproducibility.
- `--journal` (repeatable) restricts to one or more journals by ISO
  abbreviation via `[TA]`. Off by default. Use the abbreviation PubMed itself
  emits (matches the `journal` field in articles).
- Articles whose parseable page range exceeds `--max-pages` (default 50) are
  dropped and listed under `dropped_by_page_count` in the output.
- Articles without a parseable page range pass through — the PDF-level page
  check in Step 5a catches any remaining outliers.
- All applied filters are echoed back in the output's `filters` block so the
  run is reproducible from `pubmed_results.json` alone.
- A 0.12 s delay separates E-utilities calls. Bump it to ≥ 1.0 s inside the
  script if NCBI starts returning HTTP 429.

## Step 3 — Relevance scoring

Read [references/prompt-relevance-scoring.md](references/prompt-relevance-scoring.md).
Score each article 1–10 on title + abstract; keep those at **≥ 7**. Write the
surviving entries (with `doi`, `pmid`, `title` at minimum) to `shortlist.json`.

For large result sets, score in independent batches of 20–50 articles. These
batches may be run in parallel or through a provider batch-inference API
because the relevance-scoring prompt scores each article independently. Merge
the batch outputs only after every chunk returns, then apply the score
threshold.

## Step 4 — Attempt OA PDF acquisition

```bash
python3 "$SKILL_DIR/scripts/download_pdf.py" \
    --shortlist shortlist.json \
    --output-dir pdfs \
    --workers 1 \
    --pretty
```

- One subfolder per DOI under `pdfs/`, containing `paper.pdf` only when a
  compliant OA route actually yields a valid PDF.
- The downloader tries, in order: Europe PMC direct PDF for OA PMC records,
  PMC's official OA service / OA package route for OA-subset PMC records, and
  then other Unpaywall-reported OA PDF locations.
- `pdfs/download_log.json` records every attempt with status
  (`ok` / `failed` / `skipped_existing`). Cite this file from Appendix B of
  the final report.
- Reruns are cheap: existing `paper.pdf` files are skipped unless
  `--redownload` is passed.
- `--workers N` enables modest concurrent download attempts for large
  shortlists. Keep `N` small (for example 2–4) because Europe PMC, PMC, and
  Unpaywall are public services; the default remains sequential.
- `status: failed` means none of the compliant strategies produced a valid
  downloadable PDF. This includes PMC cases where a paper is visible on a PMC
  webpage but is **not** in the official PMC OA subset.
- This step is an **attempted acquisition**, not a guarantee. Some OA
  locations still fail because the returned content is an HTML interstitial,
  an access-denied page, or another non-PDF response.

## Step 5 — Extract, validate, render, and compose the report

### 5a.0. Choose an extraction budget (interactive)

Step 5a is the most expensive stage in the pipeline: each PDF drives a full
structure-extraction run. Not every shortlisted paper is downloadable, so the
real extraction universe is the **intersection of the shortlist with the
successfully acquired PDFs** — a number that is only known at runtime.

Before invoking the structure-extraction prompt, rank that intersection by
relevance score and confirm a budget with the user:

```bash
python3 "$SKILL_DIR/scripts/rank_downloaded.py" \
    --shortlist shortlist.json \
    --download-log pdfs/download_log.json \
    --pubmed-results pubmed_results.json \
    --pretty > ranked_downloaded.json
```

The script joins the three files on DOI, keeps only papers whose
`download_log.json` status is `ok` or `skipped_existing`, and sorts
descending by relevance score with publication date as the tie-breaker
(newer first — newer structures are more likely to be novel relative to
the known-warhead database).

`rank_downloaded.py` writes a machine-readable ranking to stdout and a
human-readable summary to stderr that looks like:

```
Step 4 → Step 5a bridge
Shortlisted (relevance-scoring):  126
Successfully downloaded:  46
Ranked candidates:        46
Score distribution of downloaded & ranked pool:
     score 10: 4
     score  9: 8
     score  8: 12
     score  7: 22
```

Present these numbers to the user and ask how many PDFs to extract. A
reasonable default is the top 10 by score, but this is a decision the
caller should make with full visibility of the actual download yield —
**do not silently default to "extract everything"** when the downloaded
pool is large, and do not silently extract fewer than the user expects
when it is small.

Once the budget `K` is confirmed, pass `--top-k K` to
`rank_downloaded.py` to produce the final target list, then feed only
those `K` PDFs into the structure-extraction loop below.

Structure extraction itself can also be batched or parallelized, but keep one
logical model job per PDF. This avoids cross-paper locator mixups and lets you
retry only failed PDFs. If the model provider supports batch inference, submit
the top-`K` PDF jobs as independent requests and concatenate the returned JSON
arrays only after each one passes JSON Schema validation.

### 5a. Extract candidates from acquired PDFs only

Read [references/prompt-structure-extraction.md](references/prompt-structure-extraction.md).

Before invoking the structure-extraction prompt, load the known-warhead
exclusion list for the target residue:

```python
import json
from pathlib import Path

known_path = Path(os.environ.get(
    "COVALENT_PROBE_KNOWN_WARHEADS",
    f"{os.environ['SKILL_DIR']}/data/known_warheads.json",
))
with known_path.open() as f:
    known = json.load(f)
excluded_classes = known[target_residue]  # e.g. known["Cysteine"]
```

Pass `excluded_classes` to the structure-extraction prompt as part of the
extraction context so filtering happens **before** the model emits candidates,
not after. The prompt treats that list as known and produces only novel
structures.

For each paper:

1. Use `pdfs/<doi>/paper.pdf` only when it exists.
2. Run the structure-extraction prompt with `target_residue`,
   `excluded_classes`, the paper's metadata, and the PDF path.
3. Require it to use native JSON Schema / structured-output mode with
   [references/candidate.schema.json](references/candidate.schema.json) when
   the host model API supports it.
4. Require it to emit `structure_source`, `manual_review_status`, and
   `mechanistic_family_normalized` for every candidate.
5. Append the resulting JSON objects to `candidates.json`.

Rules:

- Skip any paper whose acquired PDF is > 50 pages.
- If no valid PDF was acquired, skip that paper for extraction.
- Record acquisition failures from `pdfs/download_log.json` in Appendix B of
  the final report rather than attempting text-only extraction.

### 5b. RDKit validation and rendering

```bash
python3 "$SKILL_DIR/scripts/smiles_to_image.py" \
    --input candidates.json \
    --output candidates.enriched.json \
    --images-dir images
```

The script:

- parses every `warhead_smiles` with RDKit;
- computes `canonical_smiles`;
- renders PNGs to `images/candidate_NNN.png`;
- fills missing `mechanistic_family_normalized` values by conservative text
  normalization;
- sets new candidates to `manual_review_status: model_extracted` when the
  structure-extraction prompt omitted the field;
- marks exact duplicate structures with `duplicate_of_id`;
- sets `validation_status` to `ok`, `invalid_smiles`, or `missing_smiles`.

No cross-paper deduplication happens here — that is the report-composition
prompt's job.

Then validate the extraction contract locally:

```bash
python3 "$SKILL_DIR/scripts/validate_candidates.py" \
    --input candidates.enriched.json \
    --schema "$SKILL_DIR/references/candidate.schema.json" \
    --known-warheads "$SKILL_DIR/data/known_warheads.json" \
    --target-residue Cysteine \
    --pretty > candidate_validation.json
```

Use model-side JSON Schema / structured output when available, then run this
local validation anyway. The deterministic script checks the saved file
against `references/candidate.schema.json` and then applies workflow checks for
evidence locators, score ranges, structure locators, duplicate structures,
known-class overlaps, and candidates that remain `model_extracted` rather than
`human_confirmed`.

### 5c. Compose the academic report

Read [references/prompt-report-composition.md](references/prompt-report-composition.md).
Generate `report.md` from `candidates.enriched.json` together with the run
artifacts that carry search and acquisition metadata (`keywords.txt`,
`pubmed_results.json`, `shortlist.json`, and `pdfs/download_log.json`). The
prompt spells out the exact skeleton, ACS citation style, figure embedding
format, and what must stay in the human report versus the JSON.

Key rules recapped:

- Running prose, not bullet lists of fields.
- Superscript ACS citations `¹` inline, numbered references at the end.
- Confidence scores do NOT appear in the report body. They only influence
  where a candidate appears (body vs Appendix C) and how hedged the language
  is.
- Candidates with identical `canonical_smiles` across papers are merged in
  prose; every supporting paper is cited.

Save the final report to `report.md` in the run directory.

## Bundled scripts

- [scripts/pubmed_search.py](scripts/pubmed_search.py) — PubMed E-utilities
  search with review exclusion and page-count filtering.
- [scripts/download_pdf.py](scripts/download_pdf.py) — batch OA PDF
  acquisition attempt through Europe PMC, PMC OA service, and Unpaywall, with
  a manifest and skip-existing behaviour.
- [scripts/smiles_to_image.py](scripts/smiles_to_image.py) — RDKit validation,
  canonical SMILES, and PNG rendering.
- [scripts/validate_candidates.py](scripts/validate_candidates.py) — workflow
  validation plus JSON Schema validation for evidence locators,
  structure-source locators, manual review status, score ranges, and duplicate
  flags.
- [scripts/rank_downloaded.py](scripts/rank_downloaded.py) — joins shortlist
  with `download_log.json`, ranks downloaded papers by relevance score with
  publication-date tie-breaking, and prints a Step 4 → 5a bridge summary for
  the extraction-budget dialog.
- [scripts/run_pipeline.py](scripts/run_pipeline.py) — orchestrator for the
  deterministic steps (2, 4, 5b).

Core PDF-download modules live in [chemdeep_pdf/](chemdeep_pdf/) and are
self-contained: they prefer Europe PMC and PMC's official OA routes, then
fall back to other Unpaywall-reported OA locations.

## Known warhead database

Default location: [data/known_warheads.json](data/known_warheads.json),
shipped with the skill. Override by setting
`COVALENT_PROBE_KNOWN_WARHEADS=/path/to/your.json` or by passing
`--known-warheads` to `run_pipeline.py`.

Keyed by target residue; each value is a list of class names used as the
exclusion context passed to the structure-extraction prompt. The initial class
vocabulary is derived
from PDBcov / CovBinderInPDB and carried with local provenance metadata in the
JSON file. Current coverage:
Cysteine (69), Histidine (26), Serine (44), Lysine (24),
Aspartic Acid (10), Glutamic Acid (14), Tyrosine (7),
Threonine (15), Methionine (1).

Contributions to the exclusion list are welcome — open a pull request with
a short literature-cited rationale for any added class.

## Default parameters

- **Reaction preference:** stable covalent linkage; reversible vs irreversible
  is interpreted from the user's request and evidence in the paper.
- **Time range:** none by default; **ask the caller in Step 0** and apply via
  `--min-year` / `--max-year` for reproducibility.
- **Journal whitelist:** none by default; **ask the caller in Step 0**. If the
  caller declines, leave it unset to avoid hiding good chemistry in less
  obvious venues. Apply via repeated `--journal` flags using PubMed ISO
  abbreviations.
- **Literature scope:** chemistry-first primary literature, including
  synthetic, medicinal-chemistry, and chemical-biology methodology papers
  where transferable reaction chemistry is reported.
- **Review filter:** on (override with `--include-reviews`).
- **Max page count:** 50 (`--max-pages 0` disables).
- **Shortlist threshold:** relevance score ≥ 7.
- **Relevance-scoring batch size:** 20–50 articles per independent LLM call
  when the host supports parallel or batch inference.
- **Extraction budget default:** ask the user; suggest top 10 downloaded PDFs
  by relevance score when the pool is large.
- **Novelty filter:** the structure-extraction prompt respects
  `known_warheads.json[target_residue]`.

## Operational notes

- NCBI E-utilities enforce aggressive rate limits. A 0.12 s default delay is
  usually enough; bump to ≥ 1.0 s after any HTTP 429.
- `chemdeep_pdf/downloader.py` treats Europe PMC and PMC's OA service as the
  preferred compliant routes for PMC-backed records, and only then falls back
  to other Unpaywall-reported OA locations. Step 4 is still a retrieval
  attempt, not a guaranteed download. If none of those routes yields a valid
  PDF, the paper shows up in `download_log.json` with `status: failed` and
  should end up in Appendix B rather than silently disappearing.
- Structure drawings transcribed from PDF figures into SMILES are error-prone
  on stereochemistry and fused-ring systems. `smiles_to_image.py` catches
  syntax errors; `validate_candidates.py` checks evidence and structure
  locators; low-confidence or non-human-confirmed transcriptions are triaged
  for manual review rather than treated as publication-ready.

---
> Source: [mr-johnee/pClaw](https://github.com/mr-johnee/pClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
