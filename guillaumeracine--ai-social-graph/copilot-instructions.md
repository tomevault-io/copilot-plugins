## ai-social-graph

> This project transforms the MILA (Quebec AI Institute) researcher directory into a rich, multi-dimensional social graph with LLM-powered data enrichment.

# MILA Social Graph - Project Documentation

## Overview

This project transforms the MILA (Quebec AI Institute) researcher directory into a rich, multi-dimensional social graph with LLM-powered data enrichment.

**Current State**: 3,438 researcher profiles with enrichment pipeline operational.

## Quick Start

```bash
# 1. Install dependencies
pip install pydantic httpx anthropic openai

# 2. Confirm environment
# .env.local already holds live keys; keep it private and never commit
# (Only use .env.local.example as reference if you need field names)

# 3. Parse profiles
python scripts/parse/markdown_parser.py . data/enriched/all_profiles.json

# 4. Run enrichment
python scripts/enrich/agent_enrichment.py data/enriched/all_profiles.json --limit 100

# 5. Export graph CSVs (lightweight for LLM/CLI use)
python scripts/export/graph_edges.py data/enriched/all_profiles.json -o data/graph_export

# 6. (On demand) Export Obsidian vault markdown
python scripts/export/obsidian_export.py data/enriched/all_profiles.json -o mila_researcher_profiles
```

## Operational Playbook (Obsidian-first, Neo4j-later)
- **Parse**: Build structured profiles from markdown (`scripts/parse/markdown_parser.py`). Default excludes temp/README files.
- **Enrich**: Prefer agent pipeline (`scripts/enrich/agent_enrichment.py`) so LLM disambiguates common names; use `--no-llm` when keys are unavailable. Checkpointing stored in `data/enrichment_checkpoint.json`.
- **Quality + Fix**: Run `scripts/quality/data_quality_police.py data/enriched/all_profiles.json --fix` to flag/auto-fix self-references, missing IDs, invalid years, connectivity gaps; report saved to `data/quality_report.json`.
- **Scrape MILA pubs**: `scripts/scrape/mila_publications.py enrich --input data/enriched/all_profiles.json` to capture site-only papers and merge/dedupe.
- **Export to Obsidian**: `scripts/export/obsidian_export.py data/enriched/all_profiles.json -o mila_researcher_profiles` (or another vault folder). Open in Obsidian and use Graph View to explore the social graph prototype.
- **Export lightweight graph artifacts**: `scripts/export/graph_edges.py data/enriched/all_profiles.json -o data/graph_export` emits `graph_nodes.csv` and `graph_edges.csv` (people/labs/topics + supervision/coauthor/topic/lab edges) for LLM/CLI/Neo4j ingestion without touching thousands of `.md` files.
- Obsidian export now includes richer graph tags: membership (`member/<category>`), topics (`topic/*`), derived tags (`tag/*`), labs (`lab/*`), orgs (`org/*`), org type (`org_type/*`), and industry (`industry/*`) to improve clustering in Graph View.
- Optional index pages for Graph View: `python scripts/export/build_graph_indices.py data/enriched/all_profiles.json -o mila_researcher_profiles` creates `_topics/`, `_tags/`, `_labs/`, `_orgs/` markdown stubs linking back to members so tags show up as nodes in Obsidian.
- **Iterate**: Re-run quality + scrape after enrichment rounds; export again when satisfied. Keep `.env.local` out of git; caches live under `data/cache/`.
- **Optional qualitative insights**: use `scripts/enrich/sources/social.py` to discover social URLs (LinkedIn/Twitter/blog) from personal sites, and `scripts/enrich/sources/content.py` to fetch transcripts/blog content and produce short summaries for publications or media.
- **Derived tags for clustering**: `python scripts/analysis/tag_profiles.py data/enriched/all_profiles.json --write` adds `derived_tags` to each profile (career stage, methods, applications, labs, impact buckets). Use `--sample N` to inspect.
- **Preview a single profile (no export)**: `python scripts/export/preview_profile.py data/enriched/all_profiles.json --id <slug>` (or `--name "<Name>"`) renders to stdout.
- **JSON-only publishing**: If you need a minimal/private repository, include only `data/enriched/all_profiles.json` and your scripts; omit the Obsidian markdown exports. Set `RUN_OBSIDIAN_EXPORT=1 ./scripts/run_full_enrichment.sh` only when you explicitly want markdown.

### Qualitative Enrichment (opt-in)
- Social links: `python scripts/enrich/sources/social.py https://example.com` will print discovered social URLs from a given personal site. Integrate via `SocialEnricher.discover(name, website)` to fill `links.linkedin`/`links.twitter` and `social_media` follower/headline fields.
- Publication insights: call `summarize_publication(abstract, title)` from `scripts/enrich/sources/content.py` after fetching papers to populate `publication.summary` (1–2 sentences).
- Media/talk/blog insights: use `ContentFetcher` to grab HTML/transcripts and `ContentSummarizer` to condense into short bullets; store in `profile.content_insights` (fields: source_url, type, title, date, summary, topics).
- All helpers are cache-first under `data/cache/social` and `data/cache/content`; they run without LLM keys (fall back to extractive snippets) but improve with OpenAI/Anthropic set in `.env.local`.
- Markdown preview without export: `python scripts/export/preview_profile.py data/enriched/all_profiles.json --id <slug>` (or `--name "<Name>"`) renders a single profile to stdout. Set `RUN_OBSIDIAN_EXPORT=1 ./scripts/run_full_enrichment.sh` only when you want to regenerate all markdown files.

## Directory Structure

```
MILA_Social_Graph/
├── .env.local                 # API keys (DO NOT COMMIT)
├── CLAUDE.md                  # This file - project documentation
├── data/
│   ├── enriched/
│   │   └── all_profiles.json  # Parsed & enriched profiles
│   ├── enrichment_queue.json  # Priority-sorted processing queue
│   ├── enrichment_checkpoint.json  # Resume checkpoint
│   ├── quality_report.json    # Data quality report
│   └── cache/                 # API response caches
│       ├── semantic_scholar/
│       ├── openalex/
│       ├── arxiv/
│       ├── github/
│       └── mila_publications/
├── scripts/
│   ├── parse/
│   │   ├── models.py          # Pydantic data models
│   │   └── markdown_parser.py # Profile parser
│   ├── enrich/
│   │   ├── pipeline.py        # Basic enrichment pipeline
│   │   ├── agent_enrichment.py # LLM-powered agent enrichment
│   │   └── sources/
│   │       ├── semantic_scholar.py
│   │       ├── openalex.py
│   │       ├── arxiv.py
│   │       ├── github.py
│   │       └── labs.py
│   ├── scrape/
│   │   └── mila_publications.py # MILA website publication scraper
│   ├── quality/
│   │   └── data_quality_police.py # Data validation system
│   └── export/
│       └── obsidian_export.py # Obsidian markdown exporter
│       └── graph_edges.py # Lightweight graph nodes/edges CSV exporter
├── mila_researcher_profiles/  # Obsidian vault with enriched profiles
├── templates/                 # Obsidian templates
└── [3,438 .md files]         # Original profile markdown files
```

### Ignore rules and search hygiene
- `mila_researcher_profiles/` and `mila_researcher_profiles.bak/` are ignored in `.gitignore` and `.rgignore` to keep Git status clean and `rg` searches fast. Use `rg --no-ignore` if you need to search them explicitly.
- macOS sidecar files `._*` are also ignored.
- Regenerate the vault when needed (`obsidian_export.py`); avoid committing thousands of `.md` files.

## Data Schema

Each enriched profile contains:

```python
Profile:
  # Identity
  id: str                      # e.g., "yoshua-bengio"
  name: str                    # e.g., "Yoshua Bengio"
  membership_status: str       # e.g., "Core Academic Member"
  membership_category: enum    # core_academic, alumni, student, etc.

  # Position
  position:
    title: str                 # e.g., "Full Professor"
    institution: str           # e.g., "Université de Montréal"
    department: str

  # Research
  research_topics: list[str]   # From MILA profile
  research_keywords: list[str] # LLM-extracted (future)

  # External IDs (auto-resolved)
  external_ids:
    semantic_scholar: str      # e.g., "1751762"
    google_scholar: str        # e.g., "kukA0LcAAAAJ"
    openalex: str              # e.g., "A5086198262"
    orcid: str

  # Metrics (from APIs)
  metrics:
    h_index: int
    citation_count: int
    paper_count: int
    i10_index: int

  # Publications (from Semantic Scholar)
  publications: list[Publication]
    - id, title, year, venue, citation_count
    - authors, mila_coauthors, abstract

  # Co-authorship network
  coauthorships: list[Coauthorship]
    - author_id, author_name, is_mila
    - paper_count, first_collab_year, last_collab_year

  # Research categorization (from arXiv)
  arxiv:
    primary_category: str      # e.g., "cs.LG"
    categories: list[str]
    category_counts: dict

  # Employment / destinations
  current_org: str             # normalized employer/org name when known
  current_role: str            # role/title when known
  org_type: str                # company, startup, nonprofit, government, academic, research_lab
  org_domain: str              # inferred domain if available
  industry: str                # heuristic industry tag
  employment_history: list[EmploymentEntry]
    - organization, role, start_year, end_year, org_type, org_domain, industry, source
  # LinkedIn/GitHub hints
  links.linkedin can be fetched (best-effort) to extract headline + recent experience;
  GitHub company field is used to seed org info when available.

  # Sub-communities
  lab_affiliations: list[LabAffiliation]
    - name, short_name, pi, url

  # Open source impact (from GitHub)
  github:
    username: str
    public_repos: int
    total_stars: int
    top_repos: list[dict]
    languages: list[str]

  # Enrichment metadata
  enrichment:
    status: enum               # pending, partial, complete, failed
    sources_fetched: list[str]
    last_enriched: datetime
    confidence_score: float
```

## API Clients

### Semantic Scholar
- **Purpose**: Publications, citations, h-index, co-authors
- **Rate Limit**: 100 requests/5 min (unauthenticated)
- **Cache TTL**: 24 hours
- **Key Fields**: authorId, hIndex, citationCount, papers

### OpenAlex
- **Purpose**: Institutional affiliations, validation
- **Rate Limit**: Generous with mailto parameter
- **Cache TTL**: 24 hours
- **Key Fields**: works_count, cited_by_count, summary_stats

### arXiv
- **Purpose**: Research categories, preprint activity
- **Rate Limit**: 3 seconds between requests
- **Cache TTL**: 1 week
- **Key Fields**: primary_category, categories

### GitHub
- **Purpose**: Open source impact
- **Rate Limit**: 60/hour (unauthenticated), 5000/hour (with token)
- **Cache TTL**: 24 hours
- **Key Fields**: public_repos, stargazers_count, languages

## Enrichment Pipeline

### Basic Pipeline (`pipeline.py`)
Simple sequential enrichment without disambiguation:
```bash
python scripts/enrich/pipeline.py data/enriched/all_profiles.json -t 150
```

### Agent Pipeline (`agent_enrichment.py`)
LLM-powered enrichment with disambiguation:
```bash
# With OpenAI
export OPENAI_API_KEY="your-key"
python scripts/enrich/agent_enrichment.py data/enriched/all_profiles.json

# Without LLM (heuristic disambiguation)
python scripts/enrich/agent_enrichment.py data/enriched/all_profiles.json --no-llm
```

**Agent usage notes:**
- Needs `.env.local` (or exported env vars) for OpenAI/Anthropic; avoid referencing `.env.local.example` beyond field names.
- Uses cached API responses in `data/cache/*`; safe to keep between runs.
- Checkpointed progress stored in `data/enrichment_checkpoint.json`; reruns skip completed IDs unless you delete/update that file.
- Use `--limit` for small batches during testing; `--threshold` to focus on high-priority profiles.
- Qualitative add-ons (opt-in): call the social/content helpers after enrichment to add social links, follower counts, publication summaries, and media/talk insights. All helpers are cache-first and can run without LLMs (fallback summarization).

**Agent workflow:**
1. Build context (name, institution, supervisors, topics)
2. Search Semantic Scholar & OpenAI
3. If multiple matches → LLM disambiguation
4. Fetch papers, metrics, co-authors
5. Get arXiv categories
6. Extract lab affiliations
7. Checkpoint progress

### Priority Scoring
Profiles are processed by priority:
- Core Academic: 100 points
- Has Google Scholar ID: +25
- Has supervisors: +10
- Has students: +20
- Missing biography: +15

## MILA Publications Scraper

Scrape publications directly from MILA researcher profile pages (captures publications not in Semantic Scholar):

```bash
# Test with single profile
python scripts/scrape/mila_publications.py test --profile yoshua-bengio

# Enrich all profiles with MILA publications
python scripts/scrape/mila_publications.py enrich --input data/enriched/all_profiles.json
```

**Key Features:**
- Extracts title, date, venue, and DOI/arXiv links
- Merges with existing publications (deduplicates by title)
- 1 week cache TTL for scraped pages
- Requires: `pip install beautifulsoup4`

## Data Quality Police

Comprehensive data validation system for ensuring data accuracy, completeness, and consistency:

```bash
# Run quality checks and generate report
python scripts/quality/data_quality_police.py data/enriched/all_profiles.json -o data/quality_report.json

# Run checks AND apply auto-fixes
python scripts/quality/data_quality_police.py data/enriched/all_profiles.json --fix
```

### Quality Checks Performed

**Identity Validation:**
- Self-referential supervisors/students (auto-fixable)
- Circular supervision chains

**Completeness:**
- Missing external IDs (Semantic Scholar, OpenAlex)
- Empty publications for researchers with known paper count
- Students without supervisors
- Failed/partial enrichment

**Accuracy:**
- h-index plausibility (vs paper count)
- Citation count consistency
- Invalid publication years

**Consistency:**
- Supervisor/student exists in dataset
- Lab PI exists in dataset

**Connectivity:**
- Isolated nodes (no connections)
- Low connectivity for core academics

### Quality Report Output

Reports are saved to `data/quality_report.json` with:
- Issue counts by severity (CRITICAL, HIGH, MEDIUM, LOW)
- Issue counts by category (identity, completeness, accuracy, consistency, connectivity)
- Field coverage statistics
- Detailed issue list with suggested fixes

### Recommended Workflow

**After any data changes:**
1. Run enrichment pipeline
2. Run quality checks: `python scripts/quality/data_quality_police.py data/enriched/all_profiles.json --fix`
3. Scrape MILA publications: `python scripts/scrape/mila_publications.py enrich --input data/enriched/all_profiles.json`
4. Re-export to Obsidian: `python scripts/export/obsidian_export.py data/enriched/all_profiles.json -o profiles`

## Graph Prototyping & Neo4j Path
- **Obsidian**: Use the exported vault (`mila_researcher_profiles`) and Graph View to validate relationships (supervisors, students, MILA co-authors, labs). Regenerate markdown after each enrichment/quality round.
- **Neo4j (next)**: Plan export of nodes/edges (profiles, LAB, PAPER if needed) with relationships:
  - `SUPERVISED` / `SUPERVISES`
  - `COAUTHORED_WITH` (paper_count weight)
  - `RESEARCHES` (topic tags / arXiv categories)
  - `AFFILIATED_WITH` (lab)
  - `SIMILAR_TO` (future embeddings)
- Keep IDs stable (`profile.id` slug) for cross-tool consistency; generate CSVs or use the JSON to build import scripts when ready.

## Known Labs (Sub-communities)

The system recognizes these MILA labs:
- REAL (Montreal Robotics) - PI: Liam Paull
- RLLab (Reasoning & Learning) - PI: Doina Precup
- CCAI (Climate Change AI) - PI: David Rolnick
- CNL (Computational Neuroscience) - PI: Guillaume Lajoie
- ILLS (Language Sciences) - PI: Siva Reddy
- CRL (Chandar Research Lab) - PI: Sarath Chandar
- IVADO - Scientific Director: Aaron Courville
- SpeechBrain - PI: Mirco Ravanelli
- Duckietown - PI: Liam Paull
- Music and AI Lab - PI: Anna Huang

To add more labs, edit `scripts/enrich/sources/labs.py`.

## Enrichment Statistics (as of last run)

```
Core Academics Enriched: 56/56
Total Profiles: 3,438
With Semantic Scholar: ~60%
With arXiv papers: ~55%
With Lab Affiliations: ~40%

Top h-index:
1. Yoshua Bengio: 211
2. Hugo Larochelle: 69
3. Simon Lacoste-Julien: 47

Primary Research Areas:
- cs.LG (Machine Learning): 59%
- cs.CV (Computer Vision): 11%
- cs.RO (Robotics): 7%
- cs.CL (NLP): 7%
```

## Next Steps

### Graph Construction
Export to Neo4j with relationship types:
- SUPERVISED / SUPERVISES
- COAUTHORED_WITH (weighted by paper_count)
- RESEARCHES (topic)
- SIMILAR_TO (embedding-based)
- AFFILIATED_WITH (lab)

### Embeddings
Generate research similarity vectors:
- SPECTER2 for paper abstracts (free, local)
- OpenAI text-embedding-3-large for bios

### Visualization
- Obsidian graph view (local prototype)
- D3.js web visualization (production)

## Environment Variables

```bash
# Required for LLM disambiguation
OPENAI_API_KEY=sk-...

# Optional - higher rate limits
GITHUB_TOKEN=ghp_...
SEMANTIC_SCHOLAR_API_KEY=...

# Alternative LLM provider
ANTHROPIC_API_KEY=sk-ant-...
```

## Troubleshooting

### Rate Limiting
If you see "Rate limited" messages:
- Semantic Scholar: Wait 60 seconds, reduce batch size
- GitHub: Set GITHUB_TOKEN for 5000 req/hour
- arXiv: Already has 3-second delay built in

### Name Disambiguation
For common names (e.g., "David Wang"):
- Use LLM mode (`--no-llm` flag disables it)
- Check confidence_score in enrichment metadata
- Manual review for low-confidence matches

### Resuming Interrupted Runs
The agent pipeline saves checkpoints to `data/enrichment_checkpoint.json`.
Re-running will skip already-processed profiles.

## Contributing

1. Add new data sources in `scripts/enrich/sources/`
2. Update `Profile` model in `scripts/parse/models.py`
3. Integrate in `agent_enrichment.py`
4. Update this documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuillaumeRacine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GuillaumeRacine)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
