## distil-research

> Distil a folder of raw research files (Perplexity / ChatGPT deep research / xAI exports) into a clean, compounding knowledge base — operator-first playbook plus organised file structure with archive and quarantine layers. Use this skill whenever the user says "distil research", "distil my research", "/distil research", "clean up my research folder", "organise my research", "build a playbook from my research", or otherwise refers to processing accumulated research files in ~/Research/ or similar. Also trigger when the user mentions overlapping research files, duplicate findings across folders, or wanting a master playbook out of scattered notes. Treat ANY mention of distilling, organising, or extracting from a folder of research exports as a trigger, even without exact wording.


# Distil Research

## What this skill does

The user accumulates research files (Perplexity, ChatGPT deep research, xAI) in topic folders under `~/Research/`. Files pile up. Naming is inconsistent. Information overlaps. Most files are never reopened.

This skill converts that pile into a **compounding knowledge base** by:

1. Scanning the target folder
2. Extracting only the highest-signal **sections** (not files — see below)
3. Producing a clean, standalone **operator-first playbook** the user can re-read and reuse
4. Moving low-signal files to `_archive/`, ambiguous ones to `_quarantine/`
5. Tracking every decision in a manifest so the process is auditable and incremental

The output should feel like an operator's quick-reference written by a domain expert — not a research summary, not a literature review.

## Core principle: sections are the unit, not files

Research exports are mixed quality **inside** the same file. A Perplexity export might have one excellent tactical paragraph buried under three pages of generic best-practice padding.

Treat files as containers of sections. A single source file can end up:
- **Fully kept** — multiple high-signal sections extracted
- **Partially kept** — only 1–2 sections extracted, rest discarded
- **Archived** — even if 1 small idea was salvageable, the file as a whole is low-density
- **Quarantined** — ambiguous; needs human review before discarding

Do not treat "this file has *something* useful" as a reason to keep the whole file in the active set. Extract the signal, then move the file.

## Two modes (auto-detect from target path)

### Topic mode — frequent

Target: a single topic folder like `~/Research/<topic>/` (one folder dedicated to a single research theme). Most invocations.

### Housekeeping mode — occasional

Target: `~/Research/` itself, containing multiple topic subfolders. Different output: cross-topic overlap report + confidence-scored grades + recommendations.

## Topic mode workflow

### Step 1 — Locate the target

If the user says `/distil research` or "distil my research" without a path, ask which topic folder. Don't guess. Acceptable shortcuts: "the X one" → `~/Research/X/`.

Resolve `~` to `/Users/priyeshpatel/`.

### Step 2 — Read existing state

Look for `_distilled/manifest.json`. Present = **incremental** run. Absent = **first run**.

**Where files live (post-iter-3 structure):**

```
~/Research/[topic]/
├── <new file from Perplexity>.md   ← top level is the inbox
├── _sources/                       ← canonical kept files
├── _distilled/                     ← playbook + manifest
├── _archive/                       ← low-signal
└── _quarantine/                    ← ambiguous
```

After every run, the top level should hold only the four `_` folders. Any loose files at the top level are either (a) new files the user just dropped in, or (b) leftovers from a pre-iter-3 run that need migrating into `_sources/`.

For incremental runs:
- Scan the **top level** for new/unprocessed files (anything not inside an `_` folder)
- Scan **`_sources/`** for existing canonical files
- Hash everything found (sha256 of contents)
- Compare against `manifest.json` `files[].hash`
- Identify: new, changed, deleted, unchanged

Process only new and changed files. Unchanged files keep their existing classification.

**Migration note (one-time):** if the manifest exists but there are kept-verdict files at the top level (e.g., from a pre-iter-3 run on an existing folder), treat them as canonical and move them into `_sources/` as part of step 8. Don't reclassify; honour the existing verdict.

### Step 3 — Classify sections (tier filter)

For each new/changed file, parse into sections. For markdown, use headings (`#`, `##`, `###`) as natural boundaries. For PDFs or unstructured text, infer sections from topic shifts or treat the whole document as one section.

**Tier filter** (apply first):

- **Tier 1** — Specific tactics with concrete detail. Numbers, thresholds, named tools or platforms, step-by-step procedures, examples with real values.
- **Tier 2** — Non-obvious frameworks or mental models. Insights that change how you think about the problem class.
- **Tier 3** — Counterintuitive or contrarian findings, with reasoning.
- **Tier 4** (lower priority) — Tool / resource pointers. Only kept if clearly high-quality and non-generic.

**Discard signals** (push toward archive):
- Generic best-practice fluff
- LLM hedging ("it depends", "consider", "various factors")
- Repetition of widely-known facts
- Vague summaries without concrete grounding
- Bulleted lists that restate the section heading
- Content the user already knows because of their stated domain expertise

### Step 3.5 — Large-corpus handling (>20 files in target)

When the target folder has **more than 20 files**, full read-coverage is expensive and often unnecessary. Research export tools (Perplexity especially) produce clusters of follow-up files that share a shape: 4–7 sub-sections of generic framing with 1–2 tactical bits per file, repeating the same idea from slightly different angles.

Strategy:

1. **Identify clusters by filename pattern.** Files starting with `"How to..."`, `"Common errors in..."`, `"Examples of..."`, `"What are the most..."` on the same domain are usually drilldowns from one Perplexity research thread. Group them.

2. **Read all files in the canonical candidate set first** — large standalone files (typically RTF or files >5 KB), or files with distinctive names that don't match a cluster pattern.

3. **Sample each cluster.** Read a minimum of 1-in-3 files per cluster, and at least 2 per cluster. Use what you learn to classify the rest by pattern.

4. **Cluster-classified files get `confidence: medium`**, not high. They might contain a unique tactic you didn't see in the sample.

5. **Promote outliers to direct-read.** If a cluster-classified file has an unusual name, unusual size, or mentions specifics (tool names, version numbers, named people) not in the sampled files, read it directly before classifying.

6. **Record coverage in the manifest** under `read_coverage`:

```json
"read_coverage": {
  "directly_read": 12,
  "classified_by_pattern": 27,
  "total": 39,
  "clusters": [
    {
      "pattern": "How to X / Common errors in X / Examples of X (hook variants)",
      "size": 18,
      "sampled": 6
    },
    {
      "pattern": "Best A/B / formatting / sentence-length follow-ups",
      "size": 9,
      "sampled": 3
    }
  ]
}
```

7. **On incremental or housekeeping runs**, re-read at least 1 file from each cluster that has changed or grown. This catches new tactical content that pattern-matching would miss.

This shortcut is **only acceptable on large Perplexity-style follow-up clusters**. Do NOT use it on small corpora (<20 files) or on files of unusual shape. Default to full read-coverage; only enter cluster mode when scale demands it.

### Step 3.6 — Flag potentially fabricated technical claims

Research exports — especially Perplexity / ChatGPT deep research — can include plausible-looking technical specifics that don't actually exist. Things like a claimed GitHub repo at a specific URL, a named internal algorithm with invented component names, version numbers or release dates of internal tooling.

When a source contains technical claims that you cannot cross-verify from another kept source AND the claims read like uncited internal-engineering detail (specific URLs, repo names, file paths, APIs, internal product names), do all three:

1. **Keep the operator-tactical content** from the file (signal weights, formulas, procedures, named patterns) — those remain usable even if the surrounding framing is fabricated.
2. **Discard the uncross-verifiable technical claims.** Do not put them in the playbook.
3. **Add an entry to `hallucination_warnings`** in the manifest:

```json
"hallucination_warnings": [
  {
    "file": "X algorhythm updates.rtf",
    "concern": "Phoenix algorithm internals (Grok transformer, xai-org GitHub repo, May 15 release) — not cross-verified against public docs",
    "mitigation": "Playbook §1 keeps only the operator-tactical signal weights that align with the 2024 public X algo release"
  }
]
```

This protects the playbook from carrying forward AI-fabricated technical detail as fact, and gives the user a clear audit trail of what was filtered.

### Step 4 — Strict keep gate

A section that passes the tier filter is kept **only if it satisfies at least one** of:

1. **Unique tactic** — a specific action or procedure not already covered by another kept section.
2. **Non-overlapping framework** — a mental model that doesn't restate an existing kept framework.
3. **Materially better version** — covers the same ground as another kept section but does so with sharper detail, better example, or more accurate claim.

Sections that pass tier but fail this gate are discarded (their idea is already represented). This prevents "kept" from drifting back to "summarised". When you reject a section under this gate, log the reason against the file's `contribution_type` in the manifest.

**Canonical-source rule** (applies after the gate): when multiple kept files cover the same topical territory (e.g., "hook formulas", "long-form formatting", "thread structures"), **only one file per topic is `canonical_source`** — the most authoritative or most complete. Others with surviving sections become `partial_extract` (or `supporting_signal` if they reinforce without adding). `canonical_source` is meant to be a rare designation; if half your files end up as canonical_source, the rule isn't being applied.

**Expertise-aware rejection**: a section that would otherwise pass tier 1/2/3 still fails the gate if it covers foundational content for an area the user has stated or implied expertise in. If you can tell the user is expert in domain X (from their role, the rest of their research, prior conversation context), content that's essentially "X 101" for someone with that background is tier-4 to them, not tier-1 — regardless of how concretely the source phrases it. Classify as `low_signal`, not `partial_extract`.

### Step 5 — In-playbook deduplication pass

Before writing playbook sections, scan the set of kept sections for repeated ideas across files. When two sections cover the same idea:
- Merge into a single canonical version (best framing, best example, best detail)
- Record both source files in `playbook_sections[].sources`
- Do NOT add the second as a separate playbook section

This is distinct from the strict keep gate: the gate rejects whole sections; this pass merges overlapping content within sections that are otherwise both worth keeping.

### Step 6 — Write playbook in operator-first format

Default section shape for any **tactic or framework**:

```
## <Section name>

**Rule:** <one-line action principle — verb-led, no hedging>
**Execution:** <concrete how-to — bullets or numbered steps; ≤6 lines>
**Example:** <a real example with values, names, or numbers>
**Edge cases:** <when this breaks or doesn't apply> (optional)
```

**Sections that are inherently lists** (ranked tactics, build wedges, roadmaps, scorecards): keep the natural list structure but apply the same terseness — bullet points carry the load, prose is minimised. No padding sentences, no "in practice", no "the key thing is".

**Strategic, reflective, or observation-style sections** (Strategic Frames, Open Questions, "What's likely vs hype", "The shift to internalise"): do NOT force Rule / Execution / Example onto these — the scaffolding makes them feel mechanical. Use bulleted observations or short prose, each item one to two terse lines. Apply operator-first only where there's an actual action to take.

A reader should be able to scan a section in 15 seconds and know: what to do, how to do it, what it looks like in the wild, what could go wrong.

#### Visual formatting rules

- **Do NOT use markdown tables.** They don't render reliably across all markdown viewers (Obsidian, Notion exports, some terminal renderers). Use one of:
  - Bulleted key/value lines: `- **Signal name** — value (notes)`
  - Definition-style: `**Term:** definition`
  - YAML or JSON code blocks for genuinely structured data
- **Use ASCII diagrams when a concept benefits from visualisation.** Don't force them — but when content involves any of these, a diagram beats prose:
  - **Architectures / pipelines** — show the components and their flow
  - **Hierarchies** — three-tier systems, layers, taxonomies
  - **Sequences with branches** — request → check → branch → result
  - **Ranking visualisations** — when relative weights or sizes matter more than exact numbers
  - **State machines** — when something has clear states with transitions

Example ASCII patterns the skill should use freely:

Pipeline / flow:
```
Input ──► [ Stage 1 ] ──► [ Stage 2 ] ──► Output
                │
                └──► (side effect / log)
```

Hierarchy / layers:
```
┌─────────────────────────────────────────┐
│  Layer 3 — Analytics feedback           │
├─────────────────────────────────────────┤
│  Layer 2 — Agent pipeline               │
├─────────────────────────────────────────┤
│  Layer 1 — Research swipefile           │
└─────────────────────────────────────────┘
```

Ranking by magnitude:
```
Signal A (high-leverage)   ████████████████████████  ~24× baseline
Signal B                   ██████████████████        ~15×
Signal C                   ███████████               ~10×
Signal D                   ██                        low
```

Tree (for file structures or org charts) — already covered by the standard `├── └──` tree pattern, use freely.

Keep diagrams compact: ≤ 12 lines per diagram, ≤ 60 chars wide. If a diagram needs more, the concept is probably overloaded — split it.

### Step 7 — Two-job test (split proposal)

Quality is enforced by the strict keep gate (Step 4) and the in-playbook dedup pass (Step 5), not by an arbitrary byte cap. After writing the playbook, the only structural question left is **whether the content covers more than one distinct operational job**.

Apply the **two-job test**:

> *Does the playbook cover more than one distinct operational job — such that a user pursuing one job would have to skim past content for the other?*

Examples:
- **Two-job folder** → Job A: how to *do* a thing (tactical playbook for an active practice). Job B: how to *automate* doing that thing (the system or pipeline that makes Job A repeatable). A user pursuing Job A won't read Job B; a user building Job B won't re-read Job A. **Two-job test passes → propose split.**
- **Single-domain folder** → one job: build or operate one specific system. The user wants the full picture in one place — even at 20 KB. **Test fails → keep as one.**
- **Sub-lane folder** → one job with internal sub-lanes (e.g., two product variants in the same category). The sub-lanes inform the same operator decision, so they belong together. **Test fails → keep as one.**

If the two-job test **passes**:
- Surface a split proposal in the chat: name the candidate playbooks, give a one-sentence boundary rule per playbook, ask the user to confirm.
- Wait for user approval. Do **not** split silently.
- If approved, write split files; if user prefers single, write a single playbook.

If the two-job test **fails**:
- Write a single playbook regardless of size. `playbook_size_bytes` is recorded in the manifest as informational, not as a gate.

#### Single vs split playbook — output convention

- **Single playbook (default)**: write `_distilled/playbook.md`.
- **Split playbooks (only when user confirms the two-job test)**: write `_distilled/playbook-<slug-a>.md`, `_distilled/playbook-<slug-b>.md`, etc. Each filename uses a short kebab-case slug naming the operational job (e.g., `playbook-growth-tactics.md`, `playbook-content-os.md`). Do **not** create a `_distilled/playbook.md` file in this mode; the manifest's `playbooks` array is the index.
- **Manifest schema in split mode**: replace the top-level `playbook_size_bytes` (single value) with a `playbooks` array. Each entry: `{file, size_bytes, coverage, sections}` where `coverage` is one sentence naming the operational job this playbook serves.

The point of the two-job test: size doesn't determine whether a split is right — *operational separability* does. A 25 KB single-job playbook is fine. A 12 KB two-job playbook is worth splitting.

### Step 8 — Move files (every file gets a home)

After every run, the top level should hold only the four `_` folders. Every file gets moved into one of them based on its verdict:

- **kept** → `mv` into `_sources/` (full canonical reference)
- **partial** → see "partial routing" below
- **archived** → `mv` into `_archive/`
- **quarantined** → `mv` into `_quarantine/`

#### Partial routing — second test

A `partial` verdict means some sections were extracted into the playbook. The question is whether the *file as a whole* is still worth keeping accessible:

- **`partial` → `_sources/`** only if what remains after extraction is **itself worth re-reading** — e.g., additional context, narrative, supporting examples, or unique framing not in the playbook.
- **`partial` → `_archive/`** if the extracted sections were the only valuable content; the rest is consensus restatement or generic framing. The manifest still records the extracted sections in `sections_kept`, so traceability is preserved.

This prevents `_sources/` from drifting into "any file we touched". `_sources/` should stay reserved for files the user might actually re-open.

Create `_sources/`, `_archive/`, and `_quarantine/` if they don't exist. Never overwrite — if a same-name file already exists in the target, append `-{timestamp}` to the incoming file.

**Migration**: any file at the top level that the manifest already classifies as kept (from a pre-iter-3 run) should be moved into `_sources/` without re-classifying. Files previously classified as partial should be re-evaluated against the partial routing rule above.

**Manifest paths**: after step 8, every file entry's `path` should reflect its final location (`_sources/...`, `_archive/...`, or `_quarantine/...`).

### Step 9 — Write the manifest

Update `_distilled/manifest.json`. Schema below.

### Step 10 — Report to user

Output the end-of-run summary (format below).

## Manifest schema

### Single playbook (default)

```json
{
  "topic": "<topic-slug>",
  "last_run": "2026-05-16T...",
  "run_mode": "incremental",
  "playbook_size_bytes": 5421,
  "read_coverage": null,
  "hallucination_warnings": [],
  "files": [
    {
      "path": "_sources/perplexity-2026-04-12.md",
      "hash": "sha256:...",
      "processed_at": "...",
      "verdict": "partial",
      "contribution_type": "partial_extract",
      "reason": "one tactical section on attribution windows; rest was generic",
      "sections_kept": ["attribution-windows"],
      "sections_influenced_in_playbook": ["measurement"],
      "confidence": "high"
    }
  ],
  "playbook_sections": [...]
}
```

`playbook_size_bytes` is recorded as informational. There is no size cap — quality is enforced by the strict keep gate (Step 4) and dedup pass (Step 5), and structural decisions are made by the two-job test (Step 7).

### Split playbooks (only when the two-job test passes and the user confirms)

```json
{
  "topic": "<topic-slug>",
  "last_run": "...",
  "run_mode": "first-run",
  "playbook_size_bytes": 25372,
  "playbooks": [
    {
      "file": "_distilled/playbook-<job-a-slug>.md",
      "size_bytes": 15160,
      "coverage": "<one-sentence description of operational Job A>",
      "sections": ["1", "2", "3", "..."]
    },
    {
      "file": "_distilled/playbook-<job-b-slug>.md",
      "size_bytes": 10212,
      "coverage": "<one-sentence description of operational Job B>",
      "sections": ["1", "..."]
    }
  ],
  "read_coverage": {...},
  "hallucination_warnings": [...],
  "files": [...],
  "playbook_sections": [...]
}
```

The top-level `playbook_size_bytes` is the sum across all playbook files.

### Top-level optional fields

- `playbooks` — array. Only present in split mode (two-job test passed, user confirmed). Each entry includes a `coverage` sentence naming the operational job that playbook serves.
- `read_coverage` — `null` if every file was directly read. Object as defined in Step 3.5 if cluster mode was used.
- `hallucination_warnings` — empty array if no suspect technical claims were filtered. Array of `{file, concern, mitigation}` objects otherwise (see Step 3.6).

### `contribution_type`

A semantic label per file, complementing `verdict`:

- `canonical_source` — primary source for one or more playbook sections; few or no losses
- `supporting_signal` — reinforces points sourced primarily from other files; adds evidence or examples
- `partial_extract` — some sections are gold, rest is noise; file stays in active set if any section was kept, else moves
- `superseded_duplicate` — content fully duplicated by another kept file
- `low_signal` — generic / consensus restatement / no tier-1/2/3 content
- `ambiguous` — mixed; quarantined for review

`verdict` answers "what did we do with the file"; `contribution_type` answers "why, in semantic terms". The pair makes the decision auditable.

## Output structure

```
~/Research/[topic]/
├── _sources/                         ← canonical kept files (was top-level pre-iter-3)
│   ├── perplexity-2026-04-12.md
│   └── chatgpt-deep-research-3.md
├── _distilled/
│   ├── playbook.md                   ← operator-first, no inline citations
│   ├── manifest.json                 ← traceability + contribution types
│   └── (on --rebuild: backup-{date}/) ← previous _distilled/ snapshot
├── _archive/
│   └── chatgpt-generic-overview.md
└── _quarantine/
    └── ambiguous-mixed-notes.md
```

The top level holds **only** the four `_` folders after a run. New files dropped at the top level get processed and routed on the next `/distil research` invocation.

## Incremental vs rebuild

**Default: incremental.** Only process new and changed files. Merge into existing playbook with compression bias. Re-run the dedup pass (step 5) over the combined section set, not just the new sections.

**`--rebuild`**: full re-do. Process every file as if new. Snapshot the current `_distilled/` to `_distilled/backup-{YYYY-MM-DD-HHMM}/` first. `_archive/` and `_quarantine/` are not reset — those represent user-curated state.

**Auto-recommend rebuild** when incremental would restructure >30% of sections. Don't force it — surface as a recommendation.

## Housekeeping mode workflow

Target: `~/Research/` (the parent).

### Step 1 — Inventory

List all topic subfolders. For each, read `_distilled/playbook.md` and `_distilled/manifest.json` if present. If no `_distilled/` exists, work from raw files at the top level and (if it exists) `_sources/`.

### Step 2 — Cross-topic overlap detection

For each pair of topic folders, compare playbook sections (or raw file sections if no playbook). Identify:
- **Duplicate content** — same idea covered in two playbooks
- **Misfiled content** — a section in topic A that belongs in topic B
- **Adjacent topics that could merge** — two folders on the same domain

### Step 3 — Grade each topic with confidence

Assign A–F per topic on:
- Signal density (proportion of active content that's tier 1/2/3)
- Playbook quality (compressed, operator-first, scannable)
- Source health (ratio of active to archived files; internal duplication)

Attach a confidence label to each grade: `high` / `medium` / `low`. A folder with rich playbooks lets you grade with high confidence; a folder with only raw files (no playbook yet) typically yields medium.

### Step 4 — Filename normalisation suggestions

While scanning files, flag any whose filename is the literal Perplexity / ChatGPT prompt (e.g., `"I came accross this post . Please can you extract.md"`, `"What does this mean for X.md"`). For each, propose a structured filename using this pattern:

```
<topic>-<subtopic>-<source>-<YYYY-MM>.md
```

Where `<source>` is the LLM/tool that produced the export (`chatgpt`, `perplexity`, `xai`, `gemini`), or the author/origin if a specific person or post is the source.

Examples:
- `industry-trends-key-findings-perplexity-2026-05.md`
- `tools-evaluation-comparison-matrix-chatgpt-2026-05.md`
- `workflow-patterns-from-author-name-chatgpt-2026-05.md`

**Propose only. Never auto-rename — even at high confidence.** Filename changes can break links, scripts, and manifests. The user decides which to apply and when.

### Step 5 — Action or propose moves

**Auto-move only if ALL hold:**
1. Confidence ≥ 95% the file belongs in another topic
2. The destination topic clearly exists (subfolder present)
3. The file is not already referenced by an existing playbook

For everything else, **list in the overlap report with a confidence label** (`high` / `medium` / `low`) and wait for confirmation.

The default behaviour is "propose, don't action". Auto-moves should be exceedingly rare. Zero auto-moves is a fine outcome when everything is correctly placed.

### Step 6 — Write the overlap report

`~/Research/_overlap-report.md` with sections:

1. **Topics inventoried** — bulleted list: topic, file count, brief note (no markdown tables)
2. **Grades** — letter grade + confidence label + one-line justification per topic (bullets, not tables)
3. **Cross-topic overlap detected** — distinguish real overlaps from filename-pattern matches with different content
4. **Auto-actioned moves** — count + list (typically zero)
5. **Proposed moves** — with confidence label per item
6. **Filename normalisation suggestions** — list of proposed renames; user reviews and applies
7. **Recommendations** — non-move next steps (e.g., "run topic-mode on folder X", "consider splitting folder Y")
8. **Low-confidence calls** — surfaced explicitly so the user can override

## End-of-run summary format

After topic mode runs:

```
✓ Distilled ~/Research/[topic] ([first-run | incremental | rebuild])

Changes
- N files processed [if cluster mode used: "(X directly read, Y classified by pattern)"]
- N files kept (X full, Y partial)
- N files archived (one-line reasons)
- N files quarantined
- Playbook: N sections (size N KB)
- [if two-job test passed and split was confirmed] Split: <playbook-a> (<KB>) + <playbook-b> (<KB>)
- [if hallucination_warnings non-empty] Flagged N source(s) for potentially fabricated technical claims — see manifest hallucination_warnings

File tree
~/Research/[topic]/
├── ...

Playbook: ~/Research/[topic]/_distilled/playbook.md
Manifest: ~/Research/[topic]/_distilled/manifest.json

Next best action: <one of: review quarantine | split topic | run housekeeping | accept rename suggestions | ship>
```

After housekeeping mode runs, append the same one-line **Next best action** to the chat summary.

Pick the action that delivers the most value next, based on the run's state:

- `review quarantine` — when files were quarantined (need user judgement before they're archived or restored)
- `split topic` — when playbook size triggered the split recommendation
- `run housekeeping` — when this was a topic-mode run and the user hasn't done a cross-topic pass in a while
- `accept rename suggestions` — when housekeeping produced high-confidence filename normalisation proposals
- `ship` — when nothing else is pending: the playbook is ready to use

Keep the rest of the summary terse. The detail lives in playbook and manifest.

## Safety rules

- **Never delete files.** Move to `_archive/` or `_quarantine/` only.
- **Never overwrite files in `_archive/` or `_quarantine/`.** Append `-{timestamp}` on collision.
- **Never auto-rename files** based on normalisation suggestions. Propose only.
- **Before large batch moves** (>10 files in a single run), confirm in chat.
- **On rebuild, always snapshot** the prior `_distilled/` first.
- **Read manifest before writing it.** Never blow away prior decisions silently.

## Triggering reminders

When the user says any of these, trigger this skill:
- `/distil research`, `distil research`, `distil my research`
- "clean up my research folder", "organise my research"
- "build a playbook from [folder]", "make a playbook out of these"
- "what's in my research folder?", "any overlap in my research?"
- Any reference to processing files in `~/Research/`

If the user mentions distilling research informally during a longer conversation, surface this skill and confirm before running.

---
> Source: [noodlebindev/distil-research](https://github.com/noodlebindev/distil-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
