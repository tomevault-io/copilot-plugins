## deep-research-codex-skill

> Use when the user asks for deep research, industry research, market analysis, competitor research, evidence-backed reports, or structured web research that should produce a cited Markdown report.


# Deep Research

Generate a sourced, structured Markdown research report using staged planning, web research, file-backed intermediate artifacts, and deterministic QA scripts.

## When To Use

Use this skill for:

- Deep research on a company, industry, technology, policy, market, community, book, event, or practical topic.
- Reports that need cited sources, recent data, tables, counterarguments, and quality checks.
- Chinese-language research requests such as market research, industry reports, competitor analysis, evidence collection, and report generation.

Do not use for quick factual answers, simple web lookups, or tasks where the user only wants a short summary.

## File Layout

Resolve paths relative to this `SKILL.md`:

- `assets/profiles.json`: depth profiles: `quick`, `standard`, `deep`.
- `references/RULES.md`: report quality rules and anti-patterns. Read before drafting.
- `references/STYLE.md`: Chinese report style rules. Read before drafting Chinese reports.
- `references/TYPES.md`: classification and numbering reference. Read when the topic needs taxonomy.
- `references/prompts/outline_planning.md`: outline planning prompt reference.
- `references/prompts/task2_data_collection.md`: data collection prompt reference.
- `references/prompts/chapter_agent.md`: chapter drafting prompt reference.
- `scripts/dr_tools.py`: validation, report assembly, citation conversion, and QA.
- `reports/`: default final report directory unless the user specifies another path.

## Codex Tool Mapping

- Use the current Codex environment's available web search/open tools for current research. Browse whenever facts may have changed.
- Use shell commands to create directories, inspect files, and run Python scripts.
- Use `apply_patch` for manual edits to repository files.
- Use temp files for intermediate artifacts instead of keeping large data pools in conversation.
- If available and useful, use Scrapling as an optional local fetch enhancer through `scripts/scrapling_mcp_server.py`; do not make it mandatory.

## Operating Workflow

1. Parse the user request:
   - Topic is the remaining text after flags.
   - `-quick` => `quick`; `-deep` => `deep`; otherwise `standard`.
   - If the user specifies an output path, use it; otherwise write under `reports/`.
2. Create a run directory:
   - Windows: `$env:TEMP\dr-<random>`
   - Unix: `/tmp/dr-<random>`
   - Create `chapters/` inside it.
   - Record `start_time.txt` using the local system time.
3. Read `assets/profiles.json`, `references/RULES.md`, and `references/STYLE.md` when drafting Chinese reports.
4. Build `outline.json`:
   - Use the topic, depth profile, and `references/prompts/outline_planning.md` as guidance.
   - Include `title`, `depth_mode`, `time_anchor`, and `chapters`.
   - Each chapter needs `title`, `sections`, and `sub_questions`.
   - Write valid UTF-8 JSON to `<TMPDIR>/outline.json`.
   - Write `<TMPDIR>/task1_manifest.json` with title, chapter count, mode, and target year.
5. Collect evidence into `data-pool.json`:
   - Search the web with multiple queries derived from the outline.
   - Prefer primary sources, official statistics, company filings, standards, reputable research organizations, and authoritative news.
   - For each fact, store enough structured information for citation: source title, organization, year/date, URL, excerpt/summary, relevant chapter or sub-question, confidence, and any caveats.
   - Use recent sources when `time_anchor.mode` is `latest`; respect explicit user dates when provided.
   - Write `<TMPDIR>/data-pool.json` and `<TMPDIR>/task2_manifest.json`.
   - Run `python scripts/dr_tools.py check-datapool <TMPDIR>/data-pool.json --mode <mode>` when the structure is ready.
6. Draft chapter files:
   - Read `references/prompts/chapter_agent.md` only when drafting.
   - For each chapter, filter relevant facts from `data-pool.json`.
   - Write `<TMPDIR>/chapters/chapter-N.md`.
   - Each chapter starts with a blockquote conclusion, uses judgment-bearing headings, includes citations, and presents at least one counterpoint where relevant.
7. Validate chapters:
   - Run `python scripts/dr_tools.py validate-chapter <chapter> --expected-sections <count>`.
   - Fix or regenerate any chapter that fails.
8. Assemble and QA:
   - Run:
     `python scripts/dr_tools.py assemble-report --outline <TMPDIR>/outline.json --chapters-dir <TMPDIR>/chapters --datapool <TMPDIR>/data-pool.json --mode <mode> --target-year <year> --output <output-dir-or-file>`
   - Capture the generated report path.
   - Run:
     `python scripts/dr_tools.py convert-citations --datapool <TMPDIR>/data-pool.json <REPORT>`
   - Run:
     `python scripts/dr_tools.py qa-report <REPORT> --mode <mode> --target-year <year> --time-anchor-mode <latest|relaxed|user_specified>`
   - `qa-report` includes deterministic style lint. If it reports template terms, generic headings, or overused transition phrases, rewrite the affected passages instead of bypassing QA.
   - `qa-report` also includes citation lint. If a numeric, date, percentage, money, ranking, or specification claim lacks a numeric citation, add a source-backed citation or remove the unsupported claim.
   - When a style or citation lint fails, run `style-suggestions` or `citation-suggestions` for line-level rewrite guidance.
   - Before drafting from a completed data pool, run `source-quality <TMPDIR>/data-pool.json --mode <mode>` and fix weak or over-concentrated source pools when feasible.
   - For publication-facing reports, run `fact-support <REPORT> --datapool <TMPDIR>/data-pool.json` to confirm cited high-risk numbers appear in the evidence pool.
   - If QA fails, fix the specific reported issues and rerun.
9. Final response:
   - Provide the report path, mode, chapter count, source count, and QA result.
   - Mention any data limitations or sources that could not be accessed.

## Output Rules

- Final report is Markdown.
- Default output directory is `reports/` in this skill folder unless the user requests another location.
- Filename format: `<topic>-YYYYMMDD-HHmmss.md`.
- Preserve UTF-8 without BOM for intermediate JSON and Markdown.
- Do not include large `outline.json`, `data-pool.json`, or chapter drafts in chat unless the user asks.
- Clean temporary files only after the final report has passed QA.

## Quality Rules

Follow `references/RULES.md`. The core non-negotiables are:

- Conclusion first in every chapter.
- Every numeric claim has a traceable source.
- Include opposing or skeptical views.
- Use fact -> causality -> judgment, not generic summaries.
- Avoid filler phrases and generated-report stock language.
- Numeric, date, percentage, money, ranking, and specification claims must include numeric citations on the same line.
- Evidence pools should avoid over-reliance on one source or weak marketing/blog sources.
- Cited high-risk numbers should be present in the evidence pool, not only in the final prose.
- Generate the table of contents from `outline.json`, not from final body scraping.
- Include metadata, references, and disclaimer in the final report.

## Failure Handling

- If web access fails, report the failed source/query and continue with available authoritative sources only if the report can still meet the requested bar.
- If `dr_tools.py` crashes, rerun with the active Python executable, check file paths, and inspect the traceback before changing report content.
- If a report cannot meet QA due to insufficient sources, mark it as data-limited instead of inventing support.

---
> Source: [swllljjz/deep-research-codex-skill](https://github.com/swllljjz/deep-research-codex-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
