## paper-discovery

> Find, verify, and curate academic papers from scholarly metadata sources. Use when Codex needs to discover literature, find papers for a topic, restrict results by year/journal/ISSN, collect DOI-backed citations, compare relevance, or produce a curated bibliography for a manuscript, review, related work section, or research plan.


# Paper Discovery

## Quick Start

Use the bundled script first when the user asks for paper discovery:

```powershell
python .\scripts\discover_papers.py --query "short video multimodal user engagement health emergency" --years 2020:2026 --max-results 30 --format md --output papers.md
```

For journal-specific searches, pass the journal ISSN and name. For Information Processing & Management:

```powershell
python .\scripts\discover_papers.py --query "short video multimodal comments user perception engagement" --years 2020:2026 --issn 0306-4573 --journal "Information Processing & Management" --max-results 25 --format md --output ipm-papers.md
```

If shell network access is blocked, use available browsing/fetch tools to query the same sources manually, then follow the same curation criteria below.

## Workflow

1. Translate the user request into 2-4 concrete query variants.
   - Include domain terms: e.g. `short video`, `multimodal`, `comments`, `health information`, `emergency`, `engagement`.
   - Include method terms only when relevant: e.g. `LLM`, `NLP`, `sentiment analysis`, `computer vision`.
   - For a target journal, use ISSN over journal-name text matching.

2. Run `scripts/discover_papers.py`.
   - Use `crossref` for DOI-backed journal metadata and journal/ISSN filters.
   - Add `openalex` or `semantic_scholar` for broader discovery when the user did not restrict to a specific journal.
   - Use `--years START:END` for date limits.
   - Use `--min-score` only after checking whether the first pass is too broad.

3. Review and curate.
   - Prefer papers with a DOI, clear journal metadata, and a close conceptual match.
   - Deduplicate by DOI first, then normalized title.
   - Keep papers that help the user's argument, not just keyword matches.
   - Flag uncertain metadata, online-first records, or source conflicts.

4. Report results.
   - Include APA-style citation, DOI link, relevance reason, and how the paper can be used.
   - If the user needs manuscript integration, group papers by section: Introduction, Related Work, Methods, Discussion.
   - If the user asks for files, write a Markdown bibliography and optionally JSON metadata.

## Script Reference

`scripts/discover_papers.py` supports:

- `--query`: search query string.
- `--years`: year range such as `2020:2026`, `2024`, or `:2026`.
- `--sources`: any of `crossref`, `openalex`, `semantic_scholar`; defaults to all three unless an ISSN is supplied, where Crossref is usually best.
- `--journal`: journal name filter or preference.
- `--issn`: ISSN filter, recommended for journal-specific discovery.
- `--max-results`: maximum curated results after deduplication.
- `--min-score`: minimum local relevance score.
- `--format`: `md` or `json`.
- `--output`: output file path.

Read `references/sources.md` only when you need source-specific details or troubleshooting.

## Quality Rules

- Do not present search results as verified if metadata was not checked.
- For final bibliographies, prefer DOI URLs in `https://doi.org/...` form.
- Use publication year from the DOI metadata when source records disagree, and mention uncertainty for online-first records.
- When the target is a specific journal, verify every retained result has that journal in metadata.
- Avoid padding the list. If fewer than requested papers are truly relevant, say so and explain the search coverage.

---
> Source: [Bugliiiiii/paper-discovery](https://github.com/Bugliiiiii/paper-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
