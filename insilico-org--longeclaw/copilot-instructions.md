## longeclaw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
uv sync                                          # install deps
uv run longevityclaw                                  # interactive CLI
uv run longevityclaw --query "tell me about Horvath"  # single-query mode
uv run longevityclaw --model claude-sonnet-4-20250514 # override model
uv run longevityclaw-web                              # web UI on :8765
uv run longevityclaw-web --password secret            # with auth
```

No test suite exists. No linter is configured.

## Environment

Requires `ANTHROPIC_API_KEY` in `.env` (loaded automatically). Optional: `ANTHROPIC_MODEL`, `ANTHROPIC_BASE_URL`, `ANTHROPIC_FOUNDRY_ENDPOINT`, `ANTHROPIC_FOUNDRY_API_KEY`.

## Architecture

The system is an agent loop where Claude calls 21+ domain tools (including pathway analysis) to answer aging biology questions over a database of 233 clocks (429K coefficients, 6 modalities).

**Data flow:** User data (CSV) → `predict.py:load_beta_values()` (auto-detects modality from feature IDs) → `predict.py:run_all_applicable_clocks()` (≥95% feature coverage threshold) → `individual.py:compute_individual_profile()` (z-scores vs population) → results returned to agent.

### Core modules

| Module | Role |
|--------|------|
| `agent.py` | Conversation loop: sends messages to Claude API with tool definitions, dispatches tool calls, manages context window (truncates old rounds when >400K chars). Stores per-request traces for `/showwhy` |
| `tools.py` | 18 tool handler functions + `get_tool_definitions()` returning Anthropic tool schemas. Each `tool_*` function returns a dict serialized as tool result |
| `clock_db.py` | Singleton `ClockDatabase` loaded from `CLOCKSdata/` CSVs. Indexes clocks, coefficients, features→clocks, genes→clocks. Access via `get_db()` |
| `predict.py` | Multi-clock computation. `load_beta_values()` parses CSV (two-column or wide format). Feature importance via coefficient × value across clocks |
| `individual.py` | Population-based attribution. Three `PopulationReference` datasets (methylation 6.6K samples, transcriptomics 12.5K GTEx, proteomics 316). Z-scores, percentiles, age-binned comparisons, pathway-level aggregation |
| `train.py` | On-the-fly ElasticNet + SHAP training on hallmark pathway or custom feature sets. 5-fold CV. Uses population data as training set |
| `annotations.py` | MSigDB hallmark gene sets (50 pathways from `data/msigdb_hallmarks.gmt`), Lopez-Otin aging hallmark mappings, Fisher's exact enrichment, preranked GSEA via gseapy against MSigDB v2025.1 collections |
| `cpg_db.py` | Singleton `CpGDatabase` wrapping SQLite of 31.5K CpG genomic annotations (GRCh38). Lazy — DB opened on first query. Forward lookup by CpG ID, reverse search by chr/gene/region/island/ENCODE type |
| `gene_lookup.py` | Singleton `GeneLookup` wrapping MyGene.info via biothings_client with persistent SQLite cache (211 MB). Lazy — client created on first query. Returns name, summary, GO terms, Reactome pathways, aliases |
| `pathway_generator.py` | Pathway analysis tools: rank pathways by clock correlation, find synergies, generate hypotheses, discover aging modules, identify intervention targets |
| `pubmed.py` | NCBI E-utilities esearch+efetch. No API key needed |
| `cli.py` | Rich + prompt_toolkit UI. `@` file autocomplete, `g@` gene autocomplete, `cl@` clock autocomplete, `/` commands. Three-line live status display |
| `web/server.py` | FastAPI + WebSocket. Spawns CLI in a PTY, bridges to xterm.js in browser. Password auth with 24h session tokens. File upload to `data/uploads/` |

### Key patterns

- **Singleton DB:** `clock_db.get_db()` lazily loads and caches. All modules import it.
- **Progress callbacks:** `predict`, `individual`, `train` modules each expose `set_progress_callback()` — the agent registers callbacks that update the CLI status line.
- **Context management:** `agent.py` truncates conversation history when total chars exceed `MAX_CONTEXT_CHARS` (400K), keeping only the last `TRUNCATE_AFTER_ROUNDS` (3) assistant+tool rounds intact. First truncates old tool results to 200-char summaries, then drops oldest user+assistant pairs.
- **Ensembl mapping:** Transcriptomic clocks using Ensembl IDs (PASTA, REG) auto-map from gene symbols via `data/transcriptomic_population/ensembl_to_gene.tsv`.
- **Trace logging:** Each `chat()` call records a trace (tool calls, inputs, results, timing) in `agent.traces[]`. The CLI's `/showwhy` command renders these as a Rich tree.

### Web interface auth flow

When `--password` is set, the server stores a SHA-256 hash. `POST /api/auth` validates the password and returns a `longevityclaw_token` httponly cookie (24h TTL). The token is also injected server-side into `index.html` (replacing `__AUTH_TOKEN__`) so the JS client passes it as a query param on the WebSocket connection (`/ws/terminal?token=...`), since httponly cookies aren't accessible from JS. The PTY child process is forked with `os.fork()` + `os.execve()` — this means the web server only works on Unix.


### Data directories

- `CLOCKSdata/` — Clock definitions: `unified_aging_clocks.csv` (coefficients), `clock_descriptions.csv` (metadata)
- `data/population/` — Methylation population reference (npz + parquet stats)
- `data/transcriptomic_population/` — GTEx reference (feather + tissue stats)
- `data/proteomics_population/` — Allen Institute Olink reference (npz + stats)
- `data/msigdb_hallmarks.gmt` — 50 hallmark gene sets
- `data/cpg_annotations.sqlite` — CpG genomic annotations (31.5K CpGs, GRCh38; built by `scripts/convert_cpg_annotations.py`)
- `data/mygene_cache.sqlite` — Pre-populated MyGene.info HTTP response cache (211 MB)
- `data/msigdb/` — MSigDB v2025.1 gene set JSONs: hallmarks (50), Reactome (1787), KEGG (658), cancer (1006)
- `data/uploads/` — Web UI file uploads (gitignored)

### Adding a new tool

1. Add handler function `tool_<name>()` in `tools.py`
2. Add schema to the list in `get_tool_definitions()` (same file)
3. Add handler to dict in `get_tool_handlers()` (same file)
4. Add friendly label in `agent.py:TOOL_LABELS`

### Entry points (pyproject.toml)

- `longevityclaw` → `longevityclaw.cli:main`
- `longevityclaw-web` → `longevityclaw.web.server:main`

---
> Source: [Insilico-org/longeclaw](https://github.com/Insilico-org/longeclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
