## notebooklm-legal-research

> Legal research workflow that combines Claude's orchestration with Google NotebookLM's deep web research and AI analysis capabilities. Creates a dedicated NotebookLM notebook, runs deep multi-query legal research, performs structured IRAC/CRAC/CREAC analysis grounded in the imported sources, generates a downloadable legal memo, and optionally produces additional artifacts (slide deck, podcast, mind map, quiz, etc.). Trigger on /notebooklm-legal-research or when the user asks to research a legal topic using NotebookLM, wants a legal research notebook, or wants to combine legal research with NotebookLM artifact generation. Always asks for jurisdiction if not stated. Responds in the language of the query. Requires notebooklm CLI to be installed and authenticated.


# NotebookLM Legal Research

Orchestrate a full legal research workflow using NotebookLM for deep source discovery and analysis, with Claude managing scope, gates, and prompt sequencing.

## Prerequisites

Verify authentication before starting:
```bash
notebooklm status   # Should show "Authenticated as: email@..."
```
If not authenticated: `notebooklm login`

---

## Phase 1 — Gate 1: Scope (Claude)

Extract from the query:
- Legal question / topic
- Jurisdiction — ask if not stated or not clearly inferable
- Area of law — infer if possible; confirm only if genuinely ambiguous
- Procedural posture (litigation, transactional, advisory, academic) — infer when possible
- Report language — detect from the query itself (not from the jurisdiction). Record this explicitly. Ask only if the user writes in one language but seems to want the report in another.

Ask only what's missing. One consolidated question. Never ask more than 3 clarifying questions.

**Determine how many deep research queries are needed (3–5).** Default to 3. Add a 4th or 5th only when the issue genuinely requires it — not as a precaution. Use these signals:

- **3 queries** — single legal issue, well-defined jurisdiction, established doctrine (e.g., "Does X clause constitute unfair competition under Argentine law?")
- **4 queries** — two distinct sub-issues, or a fast-moving area where recent developments need a dedicated angle (e.g., regulatory changes, pending legislation)
- **5 queries** — multi-issue problem, multi-jurisdictional comparison, or highly specialized area where primary authorities, doctrine, enforcement practice, comparative law, and academic criticism each warrant separate coverage

For each query slot, define the angle before presenting the plan. Record the list as `RESEARCH_QUERIES` — it will be used verbatim in Phase 3.

Check for pandoc before presenting the plan:
```bash
pandoc --version 2>/dev/null && echo "pandoc: available" || echo "pandoc: not found"
```

Present a research plan and ask for confirmation:
```
Research question: [precise restatement]
Jurisdiction: [confirmed]
Area of law: [identified]
Report language: [detected from query — e.g., English, Spanish]
Research method: NotebookLM deep web research + structured IRAC/CRAC analysis
Output: Legal memo (.html + .docx)           ← if pandoc found
Output: Legal memo (.html only)              ← if pandoc not found (install: brew install pandoc)
NotebookLM notebook: will be kept for future reference

Research queries ([N] total — each runs sequentially, ~5–10 min each, ~[N×5]–[N×10] min total):
  1. [angle — e.g., primary authorities: case law, statutes, regulations]
  2. [angle — e.g., doctrine and academic analysis]
  3. [angle — e.g., specific angle tailored to the topic]
  4. [angle] ← only if N ≥ 4
  5. [angle] ← only if N = 5

⚠ Deep research queries run one at a time. More queries = broader coverage but longer wait.
  To reduce wait time, remove a query from the list above before confirming.
```
Ask: "Confirm to proceed, or adjust scope?"

---

## Phase 2 — Notebook Creation

```bash
NB_ID=$(notebooklm create "Legal Research: [Topic] — [Jurisdiction] — [YYYY-MM-DD]" --json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('id') or d.get('notebook_id') or d.get('notebook',{}).get('id',''))")
echo "Notebook ID: $NB_ID"
```
Use `$NB_ID` with `-n $NB_ID` explicitly on all subsequent commands (never rely on `notebooklm use` — parallel safety).

---

## Phase 3 — Deep Research

Run the **N queries confirmed in Phase 1** (between 3 and 5) sequentially, using the exact angles recorded as `RESEARCH_QUERIES`. Capture `research status` immediately after each query completes, before starting the next one. This is critical: `research status` only reflects the most recently completed session, so capturing after each query is the only way to collect sources from all runs.

```bash
# Query 1 — [angle from RESEARCH_QUERIES[1]]
notebooklm source add-research "[query 1 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_1_$NB_ID.json

# Query 2 — [angle from RESEARCH_QUERIES[2]]
notebooklm source add-research "[query 2 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_2_$NB_ID.json

# Query 3 — [angle from RESEARCH_QUERIES[3]]
notebooklm source add-research "[query 3 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_3_$NB_ID.json

# Query 4 — [angle from RESEARCH_QUERIES[4]] — only if N ≥ 4
notebooklm source add-research "[query 4 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_4_$NB_ID.json

# Query 5 — [angle from RESEARCH_QUERIES[5]] — only if N = 5
notebooklm source add-research "[query 5 string]" --mode deep -n "$NB_ID"
notebooklm research status --json -n "$NB_ID" > /tmp/research_5_$NB_ID.json
```

Sources are discovered but **not yet imported**. Proceed to Phase 3.5 to curate before importing.

**Jurisdiction anchoring and query language — required for every query:** Write all research query strings in the jurisdiction's primary language to retrieve sources in that language (e.g., Spanish for Argentina, Portuguese for Brazil, French for France). Always anchor with the full country name in that language (e.g., `"Argentina contrato de trabajo indemnización por despido"`, `"Brasil direito tributário ICMS"`). Deep-research results are not geographically filtered — omitting the country name reliably pulls in sources from neighbouring jurisdictions. Note: the language used here is the jurisdiction's language; this is independent of the report language, which is determined by the user's query language established in Phase 1.

Source priority guidance by jurisdiction: load `references/source-priority.md`.

---

## Phase 3.5 — Source Curation (Claude)

Load the 3 JSON files captured during Phase 3. Each file has top-level structure `{ "tasks": [ { "query": "...", "sources": [...] } ] }`.

**Apply the deduplication and filtering rules below to each batch independently first, then merge and deduplicate across batches.**

**Deduplication and filtering rules — apply in order (per batch, then cross-batch):**

1. **Drop report-only entries.** Remove any source where `result_type == 5` and `url` is empty or absent. These are the deep-research narrative chunks, not importable web pages.
2. **Drop missing URLs.** Remove any source with no `url` field or an empty `url` string.
3. **Exact-title deduplication.** Group sources with identical titles (case-insensitive, strip leading/trailing whitespace). From each group keep the one with the highest-quality domain: `.gov` / `.edu` / official court domains > `.org` > `.com` > law-firm domains. If tied, keep the first occurrence.
4. **Near-duplicate title deduplication.** Group sources whose titles share ≥80% of word tokens OR where one title is a substring of the other (after stripping punctuation). Apply the same domain-quality tie-breaking rule. One source per group.
5. **Wrong-jurisdiction filter.** Drop any source whose URL TLD or title contains a clear signal for a different country than the target jurisdiction (e.g., `.mx` when researching Argentina, `.co` when researching Spain, "Colombia" in the title when the question is about Brazil).

**Process:**
1. For each query that was run (1 through N), extract `tasks[*].sources[*]` from `/tmp/research_[i]_$NB_ID.json` → apply rules 1–5 → **Batch i curated**
2. Merge all N curated batches into one flat list → apply rules 3–4 again (cross-batch exact and near-duplicate deduplication) → **final curated list**

Present the final curated list as a compact table: `| # | Title | Domain | Batch |`

Immediately proceed to Phase 4 — do not pause for user confirmation. Notify:

> "Found [N_total] sources across [N] research runs ([Batch 1: N1, Batch 2: N2, ...]). After per-batch and cross-batch deduplication, importing [N_curated] sources now. Reply at any time to remove sources or add your own."

---

## Phase 4 — Import Curated Sources & Verify

Import each URL from the Phase 3.5 curated list. Use `source add` — this is the only path that writes sources into the notebook's queryable source store (the `IMPORT_RESEARCH` RPC used by `--import-all` only feeds the deep-research pipeline context and will not appear in `source list` or be accessible to `ask`).

```bash
# Repeat for each URL in the curated list:
notebooklm source add "<url>" -n "$NB_ID"
```

**Failure handling per URL:** If a URL returns an error (HTTP 404, access denied, timeout), log it as skipped and continue with the next URL. Do not abort the import loop on individual failures.

After all import attempts complete:

```bash
# Remove any failed imports, error pages, or stale entries
notebooklm source clean --dry-run -n "$NB_ID"
notebooklm source clean --yes -n "$NB_ID"

# Show the final source list
notebooklm source list --json -n "$NB_ID"
```

**Zero-source gate:** If the source list is empty after cleanup, do not proceed. Tell the user:
> "All sources failed to import (404s, access-blocked pages, or timeouts). Would you like to retry with different queries, add sources manually, or abort?"
Re-run Phase 3 with revised queries, or accept user-provided sources before continuing.

**Post-import wrong-jurisdiction scan:** Scan the final source list for titles or URLs that clearly indicate a different country than the target jurisdiction. Remove any found before analysis:
```bash
notebooklm source delete <source_id> -n "$NB_ID" -y
```

Present the final source list as a table (title, type, status). Notify the user and immediately continue to Phase 5 — do not wait for a reply:

> "Here are the [N] sources after cleanup. **Proceeding to analysis now** — reply at any point to add or remove sources, and I'll incorporate them before finishing."

If the user replies with sources to add or remove while analysis is in progress:
- Remove: `notebooklm source delete <source_id> -n "$NB_ID" -y`
- Add URL: `notebooklm source add "https://..." -n "$NB_ID"`
- Add file: `notebooklm source add ./file.pdf -n "$NB_ID"`
- After any addition, re-run `source clean --yes -n "$NB_ID"` before the next analysis prompt.

---

## Phase 5 — Analysis via NotebookLM Chat

Load `references/analysis-prompts.md` to select the correct framework (IRAC / CRAC / CREAC) and retrieve the prompt sequence.

**Language:** Send all prompts in the **report language established in Phase 1** (= the language the user queried in). Do NOT infer prompt language from the jurisdiction, the sources, or the research queries sent in Phase 3. A user who queries in English gets English prompts even when all sources are in Spanish; a user who queries in Spanish gets Spanish prompts. NotebookLM responds in the language of the question — prompt language directly controls the language of all analysis notes and the final report.

**If an `ask` call returns `Chat request timed out`:** retry once with a shorter prompt — remove examples/subpoints and keep only the core question. If it times out twice, split it into two follow-up messages in the same conversation.

**Multi-issue check:** If Phase 1 identified two or more distinct legal issues, switch to the **Multi-Issue Variant** defined in `references/analysis-prompts.md` — send prompts 1–4 once per issue (using note titles like `Issue-1-Rule`, `Issue-1-Application`, etc.), then the Synthesis prompt, then prompt 5 (Verification). Do not use the standard 5-prompt sequence for multi-issue cases.

Send each prompt via:

```bash
notebooklm ask "<prompt>" --save-as-note --note-title "<Section>" -n <id>
```

**Standard sequence (5 prompts):**

1. `--note-title "Issue"` — issue identification
2. `--note-title "Rule"` — governing law synthesis
3. `--note-title "Application"` — application to facts, element by element
4. `--note-title "Conclusion"` — conclusion with confidence level
5. `--note-title "Verification"` — self-reported uncertainty and gap check (mandatory)

The verification prompt (prompt 5) explicitly asks NotebookLM to flag citations it is uncertain about, identify research gaps, and summarize the strongest opposing position.

After each prompt response arrives, note the title it was saved under, then release the full response from context. Phase 5.5 retrieves the Rule and Application notes on-demand; Phase 6 retrieves all five notes on-demand.

---

## Phase 5.5 — Citation Verification

Load `references/analysis-prompts.md` (Phase 5.5 section) for the full verification query format and result classification.

**Language:** Send verification queries in the same language used for Phase 5 prompts (i.e., the query language from Phase 1).

**If a CitationCheck times out:** retry with a shorter query — ask only about one citation at a time. Add `sleep 5` between consecutive CitationCheck calls.

**Steps:**

1. Retrieve the Rule and Application notes saved during Phase 5. Note that if the Multi-Issue Variant was used, there will be multiple notes containing these keywords (e.g. `Issue-1-Rule`, `Issue-2-Application`).

```bash
notebooklm note list --json -n "$NB_ID"
# Find the IDs for ALL notes containing "Rule" or "Application" in the title, then:
notebooklm note get <note-id> -n "$NB_ID"
```

Parse each note and extract every cited source (cases, statutes, constitutional provisions, quoted passages, attributed propositions).

2. Run `notebooklm source list --json -n "$NB_ID"` and check which cited primary sources are already in the notebook.

3. For any cited primary source not in the notebook:
```bash
notebooklm source add "<actual-url-here>" -n "$NB_ID"
```
Then poll until the source status is `ready` (not `processing`) — do not use `research wait` here, as that command targets the deep-research pipeline, not individual source ingestion:
```bash
# Poll up to 300s for source to finish processing using arguments to bypass shell quote injection
python3 -c "
import subprocess, json, time, sys
nb_id = sys.argv[1]; url = sys.argv[2]; deadline = time.time() + 300
while time.time() < deadline:
    out = subprocess.check_output(['notebooklm', 'source', 'list', '--json', '-n', nb_id])
    sources = json.loads(out)
    sources = sources.get('sources', sources) if isinstance(sources, dict) else sources
    match = next((s for s in sources if url in s.get('url','') or url in s.get('title','')), None)
    if match and match.get('status') not in ('processing', ''):
        print(match.get('status')); sys.exit(0)
    time.sleep(15)
print('timeout'); sys.exit(1)
" "$NB_ID" "<actual-url-here>"
```
If the source cannot be imported (status `error`, timeout, or URL not found), mark it `[UNVERIFIED]` and skip the verification query.

4. For each citation with a source present in the notebook, send a targeted verification query:
```bash
notebooklm ask "<verification query>" --save-as-note --note-title "CitationCheck-[n]" -n <id>
```

5. Classify each result and maintain a running list using the format from `references/analysis-prompts.md` (Phase 5.5 section). For every `✓ Verified` and `~ Paraphrase — Consistent` result, copy the exact verbatim passage and location from the NotebookLM CitationCheck response into the running list. This running list feeds three places in Phase 6: the Sources Consulted verification status column, the inline block-quotes in the Legal Analysis section, and the Verification Notes section.

---

## Phase 6 — Report Assembly (Claude)

Load `references/output-templates.md` for the HTML design specification, template, and source tier classification guide.

### Step 1 — Retrieve analysis notes

Fetch the analysis notes saved during Phase 5:

```bash
notebooklm note list --json -n "$NB_ID"
```

Note the IDs for all notes generated (which may be exactly five for the standard sequence: "Issue", "Rule", "Application", "Conclusion", "Verification", or more if the Multi-Issue Variant was used). Retrieve each:

```bash
notebooklm note get <note-id> -n "$NB_ID"
```

### Step 2 — Write the HTML report

**Localization:** If the query language is not English, translate the following static strings in the template before writing — the HTML structure and CSS remain unchanged:
- Cover label: `LEGAL RESEARCH MEMORANDUM`
- Page header: `CONFIDENTIAL`
- Section headings: `Research Question`, `Legal Analysis`, `Verification Notes`, `Sources Consulted`, `Disclaimer`
- Sub-headings: `Primary Authorities`, `Secondary — Doctrine & Academic`, `Tier 3 — Law Firm & Specialized Commentary`
- Verification field labels: `Opposing position`, `Weakest link`, `Overall confidence`, `Citation mismatches`, `Unverified sources`, `Research gaps`, `Currency flags`
- Table column headers: `Source`, `Type`, `Status`
- The full disclaimer paragraph
- The footer text

Write the file to `./legal-research-[topic-slug]-[YYYY-MM-DD].html` using the **HTML Template** from `references/output-templates.md`. Fill every `<!-- PLACEHOLDER -->` comment with the actual output from Phases 5 and 5.5:

- **DOC_TITLE** — full document title (topic + jurisdiction)
- **SHORT_TOPIC** — ≤6-word header label
- **DATE_LABEL** — month and year (e.g., "April 2026")
- **JURISDICTION / AREA_OF_LAW / POSTURE / DATE / LANGUAGE / SOURCE_COUNT / CONFIDENCE** — from Phase 1 scope and source list
- **RESEARCH_QUESTION** — precise research question from Phase 1
- **ANALYSIS_CONTENT** — full IRAC/CRAC/CREAC content using the HTML patterns from `references/output-templates.md`. For every citation with status `✓ Verified` or `~ Paraphrase — Consistent`, add a `<blockquote>` immediately after the rule or application paragraph that cites it. Do **not** add block-quotes for `✗ Citation Mismatch` or `[UNVERIFIED]`.
- **Verification fields** — opposing position, weakest link, confidence, mismatches, unverified sources, gaps, currency flags — from the Phase 5 Verification note and Phase 5.5 running list
- **PRIMARY_SOURCES / SECONDARY_SOURCES / TIER3_SOURCES** — cited sources only, classified into tiers, with hyperlinks. Omit any tier section if it has no entries.

**Research Log is not included in the report.**

### Step 3 — Open / convert

Open the HTML file in the browser to verify formatting:
```bash
open ./legal-research-[topic-slug]-[YYYY-MM-DD].html        # macOS
# xdg-open ./legal-research-[topic-slug]-[YYYY-MM-DD].html  # Linux
```

To generate a `.docx`:

```bash
# Check pandoc (one-time setup)
pandoc --version 2>/dev/null || echo "Install with: brew install pandoc"

# Convert
pandoc ./legal-research-[topic-slug]-[YYYY-MM-DD].html \
  -o ./legal-research-[topic-slug]-[YYYY-MM-DD].docx \
  --from html --to docx
```

If pandoc is not available, the `.html` file opens natively in Word via File > Open.

### Step 4 — Confirm to user

```
Report saved: ./legal-research-[topic-slug]-[YYYY-MM-DD].html
DOCX:         ./legal-research-[topic-slug]-[YYYY-MM-DD].docx  (if pandoc was run)
Notebook ID:  <id>  (kept for future reference and artifact generation)
```

---

## Phase 7 — Additional Artifacts

Ask the user:
> "The report has been downloaded. Would you like to generate any additional outputs from this notebook?"

Present the available options:
- **Podcast** (`generate audio`) — deep-dive audio overview
- **Slide Deck** (`generate slide-deck`) — downloadable as PDF or PPTX
- **Mind Map** (`generate mind-map`) — hierarchical concept map (instant, downloads as JSON)
- **Quiz** (`generate quiz`) — downloadable as JSON, Markdown, or HTML
- **Flashcards** (`generate flashcards`)
- **Infographic** (`generate infographic`)
- **Video** (`generate video`)

Generate and download whatever the user selects. For long-running generations (audio, video, slide deck, quiz), use `TaskCreate` with the subagent pattern:

```bash
# Trigger generation (example: audio)
notebooklm generate audio -n <id>

# Poll for completion — verify the exact command with `notebooklm --help` or `notebooklm generate --help`,
# as the wait/download syntax may vary by CLI version:
notebooklm artifact wait <artifact-id> -n <id> --timeout 1800
# OR: notebooklm generate wait -n <id> --timeout 1800
# Download once ready:
notebooklm artifact download <artifact-id> -n <id>
```

Check `notebooklm --help` at runtime to confirm the correct wait and download command for the installed CLI version.

---

## Reference Files

| File | Load when |
|------|-----------|
| `references/analysis-prompts.md` | Phase 5 — selecting framework and retrieving prompt sequence; Phase 5.5 — citation verification query format |
| `references/source-priority.md` | Phase 3 — designing research queries by jurisdiction |
| `references/output-templates.md` | Phase 6 — report assembly: HTML template, source tier classification, type labels |

---
> Source: [Flosters/notebooklm-legal-research](https://github.com/Flosters/notebooklm-legal-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
