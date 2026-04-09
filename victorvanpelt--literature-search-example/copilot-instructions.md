## literature-search-example

> This project supports a **master thesis literature search** in management, business, finance, accounting, and economics.

# Literature Search Project

This project supports a **master thesis literature search** in management, business, finance, accounting, and economics.

## Project Structure

- `files/` — Skill reference files (journal tiers, search techniques, hallucination prevention)
- `results/` — Search results and Excel summaries
- `.claude/agents/` — Specialized agents for paper finding, summarization, and auditing

## Key Principles

1. **Never fabricate citations.** Every paper must be verified via web search. See `files/hallucination-prevention.md`.
2. **Use the tier system.** Papers are ranked FT50 > ABS 4*/ERIM STAR > ABS 4/ERIM P > ABS 3 > SSRN/Working Papers. See `files/journal-tiers.md`.
3. **Minimum 5 searches per topic.** Follow the search protocol in `files/search-techniques.md`.
4. **Excel output.** Results go into `results/literature_search.xlsx` with columns: Year, Title, Authors, Journal, Tier, Abstract, Citations, Link, Relevance, Verified.
5. **Audit trail.** Every search session should be auditable — the auditor agent checks all entries.

## Agents

Three specialized agents are available via `/paper-finder`, `/summarizer`, and `/auditor`:

- **paper-finder** — Executes the multi-round search protocol, finds and verifies papers
- **summarizer** — Takes found papers and writes/updates the Excel spreadsheet
- **auditor** — Reviews all entries in the spreadsheet for hallucination, completeness, and tier accuracy

## Workflow

1. User describes a research topic or question
2. Run the `paper-finder` agent to search and verify papers
3. Run the `summarizer` agent to add results to the Excel file
4. Run the `auditor` agent to validate all entries
5. Iterate — refine search terms, fill gaps, expand to adjacent topics

## Excel File Format

The file `results/literature_search.xlsx` has these columns:

| Column | Description |
|--------|-------------|
| Year | Publication year (or "Working Paper" for unpublished) |
| Title | Exact title from search results |
| Authors | Author names from search results |
| Journal | Full journal name from search results |
| Tier | FT50 / ABS 4* / ERIM STAR / ABS 4 / ERIM P / ABS 3 / SSRN / Unranked |
| Abstract | Abstract text from search results or fetched page |
| Citations | Approximate citation count if available |
| Link | URL to the paper (Semantic Scholar, journal, or SSRN) |
| Relevance | 1-2 sentence summary of relevance to the research topic |
| Verified | YES/NO — whether metadata was confirmed via fetch |
| Search_Session | Identifier for which search session found this paper |
| Notes | Any additional notes (e.g., "year not confirmed") |

## Dependencies

- Python 3 with `openpyxl` for Excel file creation/editing

## Common Commands

```bash
# View current results
python3 -c "import openpyxl; wb=openpyxl.load_workbook('results/literature_search.xlsx'); ws=wb.active; [print(row) for row in ws.iter_rows(max_row=5, values_only=True)]"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/victorvanpelt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/victorvanpelt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
