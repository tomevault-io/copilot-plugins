## zotero-skill

> This repository is a local Codex-managed toolkit for:

# AGENTS.md — Zotero Research Automation for Codex

## Purpose

This repository is a local Codex-managed toolkit for:

1. Searching scholarly papers by topic.
2. Using safe public academic metadata APIs instead of scraping Google Scholar or publisher websites.
3. Deduplicating and ranking papers.
4. Importing selected references into Zotero through the Zotero Web API.
5. Creating a reusable Codex Skill at `skills/zotero_research/SKILL.md`.

The expected user workflow is:

```bash
python research_to_zotero.py --topic "retrieval augmented generation evaluation" --limit 20 --since 2020 --dry-run
python research_to_zotero.py --topic "retrieval augmented generation evaluation" --limit 20 --since 2020 --collection "RAG Evaluation"
```

Always prefer a dry run before writing to Zotero.

---

## Repository Context

The user already has:

- Ubuntu installed.
- Zotero desktop installed on Ubuntu.
- Zotero browser connector installed.
- A Zotero account with sync enabled or planned.
- A Zotero Web API key with write access.
- Existing project files:
  - `README.md`
  - `requirements.txt`
  - `zotero_add.py`

Codex should extend this project rather than replacing it blindly.

---

## Non-Negotiable Safety Rules

1. Never print, log, commit, or expose the Zotero API key.
2. Never hard-code API keys in source files.
3. Use environment variables and optionally `.env`.
4. Add `.env` to `.gitignore`.
5. Do not scrape Google Scholar.
6. Do not bypass publisher paywalls.
7. Do not download paywalled PDFs.
8. Do not mass-import hundreds of papers without a dry run.
9. Always deduplicate before importing.
10. If metadata quality is poor or uncertain, show the paper in dry-run output but do not auto-import it unless the user explicitly requests it.

---

## Target Architecture

```text
User topic
  ↓
research_to_zotero.py
  ↓
search_papers.py
  ↓
OpenAlex / Crossref / arXiv / Semantic Scholar
  ↓
Normalized paper objects
  ↓
Deduplication and ranking
  ↓
zotero_add.py
  ↓
Zotero Web API
  ↓
Zotero desktop sync
```

The implementation should use public scholarly metadata APIs:

- OpenAlex: default broad scholarly search.
- Crossref: DOI metadata enrichment.
- arXiv: preprints for CS, math, physics, statistics, quantitative biology, quantitative finance, and electrical engineering.
- Semantic Scholar: optional, useful for citation count and relevance metadata.

---

## Required Files

Codex should create or update the following files:

```text
README.md
requirements.txt
.env.example
.gitignore
search_papers.py
zotero_add.py
research_to_zotero.py
skills/zotero_research/SKILL.md
```

Optional but recommended:

```text
tests/
tests/test_normalize.py
tests/test_deduplicate.py
```

---

## Python Dependencies

`requirements.txt` should include at least:

```txt
pyzotero
requests
python-dotenv
feedparser
rapidfuzz
rich
```

Optional test dependency:

```txt
pytest
```

Purpose of dependencies:

- `pyzotero`: interact with Zotero Web API.
- `requests`: call academic metadata APIs.
- `python-dotenv`: load local `.env`.
- `feedparser`: parse arXiv Atom API responses.
- `rapidfuzz`: fuzzy title deduplication.
- `rich`: clear terminal output.

---

## Environment Variables

Create `.env.example`:

```env
ZOTERO_USER_ID=your_zotero_user_id
ZOTERO_API_KEY=your_zotero_api_key
ZOTERO_LIBRARY_TYPE=user
ZOTERO_COLLECTION_NAME=Auto Imported Papers

# Optional
SEMANTIC_SCHOLAR_API_KEY=
OPENALEX_MAILTO=your_email@example.com
```

`ZOTERO_LIBRARY_TYPE` should usually be `user`.

For group libraries, allow the user to set:

```env
ZOTERO_LIBRARY_TYPE=group
ZOTERO_GROUP_ID=your_group_id
```

but group support can be secondary.

---

## `.gitignore` Requirements

Ensure `.gitignore` contains:

```gitignore
.env
.venv/
__pycache__/
*.pyc
.pytest_cache/
```

---

## Common Paper Schema

All search sources must be normalized into this schema:

```python
{
    "title": str,
    "authors": [
        {"firstName": str, "lastName": str}
    ],
    "year": str | None,
    "date": str | None,
    "doi": str | None,
    "url": str | None,
    "abstract": str | None,
    "publication_title": str | None,
    "source": str,
    "source_id": str | None,
    "arxiv_id": str | None,
    "openalex_id": str | None,
    "semantic_scholar_id": str | None,
    "citation_count": int | None,
    "item_type": "journalArticle" | "preprint" | "conferencePaper" | "bookSection" | "book"
}
```

Rules:

- DOI should be normalized to lowercase and stripped of URL prefixes.
- Titles should be stripped and normalized for deduplication.
- Authors should use Zotero creator dictionaries.
- If first/last name parsing is uncertain, put the full name in `lastName` and leave `firstName` empty.
- Prefer `journalArticle` unless there is a clear reason to use another Zotero item type.
- arXiv papers can be stored as `preprint` if the Zotero API template supports it; otherwise use `journalArticle` with arXiv metadata in `extra`.

---

## `search_papers.py` Requirements

Implement these functions:

```python
def search_openalex(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def search_crossref(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def search_arxiv(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def search_semantic_scholar(topic: str, limit: int = 20, since: int | None = None) -> list[dict]:
    ...

def enrich_with_crossref_by_doi(paper: dict) -> dict:
    ...

def normalize_paper(raw: dict, source: str) -> dict:
    ...

def search_sources(topic: str, sources: list[str], limit: int, since: int | None = None) -> list[dict]:
    ...
```

### OpenAlex

Use OpenAlex as the default source.

Recommended request pattern:

```text
https://api.openalex.org/works?search=<topic>&per-page=<limit>
```

If `OPENALEX_MAILTO` is set, include it in API parameters when supported.

Prefer fields:

- `display_name`
- `authorships`
- `publication_year`
- `publication_date`
- `doi`
- `id`
- `cited_by_count`
- `primary_location`
- `locations`
- `abstract_inverted_index`
- `host_venue` or source equivalent if available

OpenAlex abstracts may be in inverted-index format. Implement a helper:

```python
def restore_openalex_abstract(inverted_index: dict | None) -> str | None:
    ...
```

### Crossref

Use Crossref for:

- General search fallback.
- DOI enrichment when a DOI exists.

Normalize:

- `title`
- `author`
- `published-print`, `published-online`, `issued`
- `DOI`
- `container-title`
- `URL`

### arXiv

Use arXiv API via `feedparser`.

Recommended query pattern:

```text
http://export.arxiv.org/api/query?search_query=all:<topic>&start=0&max_results=<limit>&sortBy=relevance&sortOrder=descending
```

Normalize:

- title
- authors
- published date
- summary as abstract
- arXiv ID
- PDF URL only if open and directly provided by arXiv
- tags/categories into note or extra

### Semantic Scholar

Use Semantic Scholar only if available or if unauthenticated usage works without rate problems.

Normalize:

- `paperId`
- `title`
- `authors`
- `year`
- `abstract`
- `venue`
- `citationCount`
- `externalIds`, especially DOI and ArXiv
- `url`

If `SEMANTIC_SCHOLAR_API_KEY` exists, use it through the documented header.

---

## Deduplication Requirements

Implement in `research_to_zotero.py` or a helper module:

```python
def normalize_doi(doi: str | None) -> str | None:
    ...

def normalize_title(title: str) -> str:
    ...

def deduplicate_papers(papers: list[dict]) -> list[dict]:
    ...
```

Deduplication order:

1. Exact DOI match.
2. Exact arXiv ID match.
3. Exact OpenAlex ID match.
4. Exact Semantic Scholar ID match.
5. Fuzzy title matching with `rapidfuzz`.

Suggested fuzzy title threshold:

```python
FUZZY_TITLE_THRESHOLD = 92
```

When duplicates are found, merge them:

- Prefer non-empty DOI.
- Prefer richer abstract.
- Prefer higher citation count.
- Combine source IDs.
- Preserve all source names in a field like `sources_seen`.

---

## Ranking Requirements

Implement:

```python
def rank_papers(papers: list[dict], topic: str) -> list[dict]:
    ...
```

Ranking signals:

1. Topic relevance based on title/abstract fuzzy match.
2. Publication recency.
3. Citation count.
4. DOI availability.
5. Abstract availability.
6. Trusted source metadata completeness.

Keep the ranking simple and transparent. Do not invent relevance scores unless calculated.

---

## `zotero_add.py` Requirements

Preserve any existing useful functions in `zotero_add.py`, but add or ensure these functions:

```python
def get_zotero_client():
    ...

def get_or_create_collection(collection_name: str) -> str:
    ...

def search_existing_by_doi(doi: str) -> list[dict]:
    ...

def search_existing_by_title(title: str) -> list[dict]:
    ...

def paper_to_zotero_item(paper: dict) -> dict:
    ...

def create_zotero_item(paper: dict, collection_key: str | None = None) -> dict:
    ...

def add_note_to_item(item_key: str, note_html: str) -> dict:
    ...
```

### Zotero Item Mapping

Map normalized paper fields to Zotero fields:

```python
item["title"] = paper["title"]
item["creators"] = paper["authors"]
item["date"] = paper["date"] or paper["year"]
item["DOI"] = paper["doi"]
item["url"] = paper["url"]
item["abstractNote"] = paper["abstract"]
item["publicationTitle"] = paper["publication_title"]
```

Tags:

```python
[
    {"tag": "auto-imported"},
    {"tag": f"source:{paper['source']}"},
    {"tag": f"topic:{topic}"}
]
```

Extra field should include available IDs:

```text
OpenAlex: ...
Semantic Scholar: ...
arXiv: ...
Sources seen: ...
```

### Zotero Duplicate Check

Before creating an item in Zotero:

1. If DOI exists, search Zotero by DOI.
2. Search Zotero by normalized/fuzzy title.
3. If likely duplicate exists, skip import and report it.

Never create duplicate items knowingly.

---

## `research_to_zotero.py` CLI Requirements

Use `argparse`.

Required argument:

```bash
--topic
```

Supported arguments:

```bash
--limit
--since
--collection
--sources
--min-citations
--dry-run
--yes
--sort
```

Defaults:

```python
limit = 20
sources = ["openalex"]
dry_run = True
sort = "relevance"
```

Important behavior:

- Default to dry-run unless `--yes` or no `--dry-run` policy is intentionally changed.
- Print a clear table of candidate papers before import.
- Show title, year, DOI/arXiv, source, citations, and URL.
- If importing, print imported item keys and skipped duplicates.
- Return non-zero exit code for serious errors.
- Return zero if no papers found but the process itself succeeded.

Example commands:

```bash
python research_to_zotero.py --topic "graph neural networks for drug discovery" --limit 20 --since 2020 --dry-run
```

```bash
python research_to_zotero.py --topic "graph neural networks for drug discovery" --limit 20 --since 2020 --collection "GNN Drug Discovery" --sources openalex arxiv semantic_scholar --yes
```

```bash
python research_to_zotero.py --topic "large language model agents" --limit 30 --since 2021 --min-citations 10 --collection "LLM Agents" --yes
```

---

## Child Note Format

Each imported Zotero item should receive a child note like this:

```html
<h2>Auto-import metadata</h2>
<p><b>Search topic:</b> ...</p>
<p><b>Source API:</b> ...</p>
<p><b>Imported at:</b> ISO_TIMESTAMP</p>
<p><b>DOI:</b> ...</p>
<p><b>arXiv ID:</b> ...</p>
<p><b>OpenAlex ID:</b> ...</p>
<p><b>Semantic Scholar ID:</b> ...</p>
<h3>Abstract</h3>
<p>...</p>
<h3>Reason for inclusion</h3>
<p>Matched the search topic by title/abstract and passed configured filters.</p>
```

Do not include secrets in notes.

---

## `skills/zotero_research/SKILL.md`

Codex should create this file exactly or very close to this content:

```markdown
# Zotero Research Skill

Use this skill when the user wants to search academic papers by topic, deduplicate them, rank them, and save selected references to Zotero.

## Goal

Given a research topic, search scholarly metadata sources, deduplicate results, rank papers, and import selected papers into Zotero.

## Environment

Required environment variables:

- ZOTERO_USER_ID
- ZOTERO_API_KEY
- ZOTERO_LIBRARY_TYPE

Optional environment variables:

- ZOTERO_GROUP_ID
- SEMANTIC_SCHOLAR_API_KEY
- OPENALEX_MAILTO

Never print, log, or commit API keys.

## Default workflow

1. Parse the user's topic.
2. Search OpenAlex first.
3. Search arXiv if the topic is related to computer science, mathematics, physics, statistics, quantitative biology, quantitative finance, or electrical engineering.
4. Search Semantic Scholar if available.
5. Use Crossref to enrich DOI metadata when a DOI is present.
6. Normalize all results into a common paper schema.
7. Deduplicate by DOI, arXiv ID, OpenAlex ID, Semantic Scholar ID, and fuzzy title matching.
8. Rank papers by:
   - topical relevance
   - publication year
   - citation count
   - source quality
   - availability of DOI or arXiv ID
9. Before importing, search Zotero for existing DOI or title matches.
10. Create or reuse a Zotero collection named after the topic.
11. Import selected papers into Zotero.
12. Add tags:
   - auto-imported
   - source:<source>
   - topic:<topic>
13. Add a child note with:
   - search topic
   - source API
   - abstract
   - reason for inclusion
   - imported timestamp

## Commands

Dry run:

```bash
python research_to_zotero.py --topic "<topic>" --limit 20 --dry-run
```

Import:

```bash
python research_to_zotero.py --topic "<topic>" --limit 20 --collection "<collection>" --yes
```

Recent papers:

```bash
python research_to_zotero.py --topic "<topic>" --limit 30 --since 2021 --sort relevance --dry-run
```

## Safety and quality rules

- Always run a dry run first unless the user explicitly asks to import immediately.
- Do not scrape Google Scholar.
- Do not download paywalled PDFs.
- Do not bypass publisher access controls.
- Prefer official public APIs.
- Do not create duplicate Zotero items.
- If metadata confidence is low, print the candidate but do not import it automatically.
```

---

## README Requirements

Update `README.md` with:

1. Project purpose.
2. Setup instructions.
3. How to create `.env`.
4. How to install dependencies.
5. Dry-run examples.
6. Import examples.
7. Zotero sync explanation.
8. Safety notes:
   - no Google Scholar scraping
   - no paywall bypass
   - no API key in Git
9. Troubleshooting:
   - invalid API key
   - collection creation failed
   - no papers found
   - duplicates skipped
   - arXiv rate issues
   - Semantic Scholar rate limits

---

## Expected Codex Implementation Plan

When asked to implement this repository, Codex should:

1. Inspect existing files:
   - `README.md`
   - `requirements.txt`
   - `zotero_add.py`
2. Preserve useful existing code.
3. Add missing dependencies.
4. Create `.env.example`.
5. Update `.gitignore`.
6. Implement `search_papers.py`.
7. Refactor or extend `zotero_add.py`.
8. Implement `research_to_zotero.py`.
9. Create `skills/zotero_research/SKILL.md`.
10. Update README.
11. Run basic syntax checks:

```bash
python -m py_compile search_papers.py zotero_add.py research_to_zotero.py
```

12. If tests are created, run:

```bash
pytest
```

13. Do not run real imports unless the user explicitly provides environment variables and requests import.

---

## Acceptance Criteria

The implementation is acceptable if:

1. `python -m py_compile search_papers.py zotero_add.py research_to_zotero.py` passes.
2. `python research_to_zotero.py --topic "test topic" --limit 5 --dry-run` runs without writing to Zotero.
3. The program can search OpenAlex and print normalized results.
4. The program can deduplicate by DOI and title.
5. The program refuses to import without Zotero credentials.
6. The program imports into Zotero only when explicitly requested.
7. Imported papers are placed into the requested collection.
8. Imported papers receive tags and a child note.
9. Duplicate Zotero items are skipped.
10. No secrets are printed or committed.

---

## User-Facing Codex Prompt

The user can paste this prompt into Codex:

```text
请读取当前项目的 AGENTS.md，并按其中的要求实现“给定主题自动搜索论文并导入 Zotero”的工具。

请先检查现有 README.md、requirements.txt、zotero_add.py，不要盲目覆盖已有代码。

请创建或更新：
- requirements.txt
- .env.example
- .gitignore
- search_papers.py
- zotero_add.py
- research_to_zotero.py
- skills/zotero_research/SKILL.md
- README.md

实现后请运行：
python -m py_compile search_papers.py zotero_add.py research_to_zotero.py

先不要真实导入 Zotero。请只运行一个 dry-run 示例：
python research_to_zotero.py --topic "retrieval augmented generation evaluation" --limit 5 --dry-run

注意：
- 不要爬 Google Scholar。
- 不要下载付费 PDF。
- 不要泄露 API key。
- 默认 dry-run。
- 写入 Zotero 前必须查重。
```

---

## Notes for Future Maintenance

Good future improvements:

1. Add BibTeX export.
2. Add CSV import/export.
3. Add OpenAlex semantic search mode.
4. Add topic-specific query expansion.
5. Add an allowlist or blocklist of venues.
6. Add citation-count based ranking.
7. Add optional open-access PDF attachment only when the PDF is legally open and directly linked.
8. Add a `--review` mode where the user selects paper numbers before import.
9. Add tests for DOI normalization and OpenAlex abstract restoration.
10. Add support for Zotero group libraries.

---
> Source: [zjjues/zotero-skill](https://github.com/zjjues/zotero-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
