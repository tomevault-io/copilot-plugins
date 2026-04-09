## wiki-graph

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Critical Rules

- **NEVER generate fake, simulated, or placeholder data.** All data must come from real wiki dumps, real embeddings, and real LLM responses. If a data source is unavailable (e.g. Ollama is down), fail gracefully with a clear error -- do not substitute fake results.
- **NEVER use fallback/mock data** for embeddings, cluster names, article content, or any other pipeline output. If something fails, report the failure.

## Project Overview

MediaWiki Graph Analysis Platform - a toolkit for analyzing any MediaWiki-based wiki using semantic search, knowledge graph extraction, network visualization, and LLM embeddings. Works with Fandom/Wikia, Wikipedia, or self-hosted MediaWiki dumps.

## Commands

### Setup (First Time)

```bash
# Edit setup.py first to configure your wiki's dump URL, then:
python setup.py
```

Requires: `wget`, `p7zip-full` (`sudo apt install p7zip-full`)

### Generate Visualizations

```bash
python generate_graphs.py                           # Full run (HDBSCAN eom, auto cluster count)
python generate_graphs.py --test                    # Quick test (100 articles)
python generate_graphs.py --limit 500               # Limit to 500 articles
python generate_graphs.py --clustering kmeans --clusters 30  # K-Means with 30 clusters
python generate_graphs.py --min-cluster-size 50     # HDBSCAN with smaller min cluster size
python generate_graphs.py --cluster-selection leaf   # Granular clusters (breaks up mega-clusters)
python generate_graphs.py --umap-dim 5              # 5D UMAP reduction before clustering
python generate_graphs.py --wiki-name "Memory Alpha" # Custom name for titles/filenames
python generate_graphs.py --name-clusters           # Use LLM for cluster names (requires Ollama)
python generate_graphs.py --model 4B                # Use larger model (default: 0.6B)
python generate_graphs.py --text-weight 0.8         # Weight text vs graph embeddings
python generate_graphs.py --rebuild                 # Force rebuild all embeddings
CUDA_VISIBLE_DEVICES=1 python generate_graphs.py    # Use specific GPU
```

### Full Run (CPU-Safe)

For large datasets (10K+ articles), always use the run script which includes CPU limits to prevent machine freeze:

```bash
./run_star_trek.sh                  # Full run with LLM cluster naming
./run_star_trek.sh --test           # Quick test (100 articles)
./run_star_trek.sh --limit 1000     # Limit to 1000 articles
./run_star_trek.sh --skip-setup     # Skip download/extraction step
```

The script wraps `setup.py` (download + extraction) then `generate_graphs.py` with `taskset -c 0-5`, `nice -n 19`, and `OMP_NUM_THREADS=2` to limit the process to 6 of 14 CPU cores. Full 59K articles takes ~90 min with this config.

## Architecture

### Data Flow

```
MediaWiki XML dump --> setup.py --> Markdown files (data/pages/)
                                       |
                              generate_graphs.py
                                       |
              +------------------------+------------------------+
              |                                                 |
    Text embeddings (Qwen3, GPU)                    Graph embeddings (node2vec, CPU)
              +------------------------+------------------------+
                                       |
                    Hybrid embeddings (weighted combination)
                                       |
                         UMAP reduction (10D for clustering)
                                       |
                    HDBSCAN (default) or K-Means clustering
                                       |
                    Cluster quality metrics (silhouette, CH, DB)
                                       |
                    LLM cluster naming (Ollama qwen3:8b, optional)
                                       |
                              Visualizations (outputs/)
```

### Key Functions in generate_graphs.py

| Function | Purpose |
|----------|---------|
| `parse_wiki_markup()` | Parse `[[...]]` links and clean MediaWiki markup |
| `load_articles_with_links()` | Load articles and build link graph |
| `generate_text_embeddings_with_cache()` | GPU text embeddings with SQLite cache |
| `generate_graph_embeddings()` | node2vec on wiki link network |
| `combine_embeddings()` | Weighted concatenation (default 70% text, 30% graph) |
| `reduce_dimensions()` | UMAP reduction to N dimensions before clustering |
| `cluster_hdbscan()` | HDBSCAN with auto cluster count and noise reassignment |
| `discover_clusters()` | K-means clustering (alternative to HDBSCAN) |
| `evaluate_clusters()` | Silhouette, Calinski-Harabasz, Davies-Bouldin metrics |
| `get_cluster_keywords()` | TF-IDF keyword extraction -- finds words distinctive to each cluster |
| `get_cluster_names_via_llm()` | LLM cluster naming via Ollama (qwen3:8b, `think=False` required) |
| `generate_distinct_colors()` | Extended color palette (tab20 + golden-ratio HLS) for 20+ clusters |
| `_label_cluster_centroids()` | Smart centroid labels for scatter plots |
| `_label_representative_points()` | Per-cluster representative labels with adjustText overlap prevention |

### Clustering Pipeline

The pipeline follows best practices from semantic clustering research:

1. **UMAP reduction** (10D default) -- clustering in reduced space significantly outperforms raw high-dimensional clustering
2. **HDBSCAN** (default) -- auto-discovers cluster count from data density, labels noise points, reassigns them to nearest centroid
   - `--cluster-selection eom` (default): Fewer, larger clusters. "Excess of mass" merges sub-clusters. 59K articles -> ~15 clusters (one 70% mega-cluster)
   - `--cluster-selection leaf`: More granular clusters at the leaves of the condensed tree. 59K articles -> ~108 clusters (largest 3.4%), much better topic separation
3. **K-Means** (alternative via `--clustering kmeans`) -- requires specifying cluster count, assigns all documents
4. **Quality metrics** -- silhouette score (>0.5 good), Calinski-Harabasz (higher better), Davies-Bouldin (lower better)
5. **Visualization** -- scatter plots use separate 2D projections from full embeddings (not the 10D reduced space)

### Embedding Models

- **0.6B (default)**: `Qwen/Qwen3-Embedding-0.6B` - 1024 dimensions, stable
- **4B**: `Qwen/Qwen3-Embedding-4B` - 2560 dimensions, better but may crash GPUs

### Output Files

Generated in `outputs/` with `{wiki_slug}_` prefix:
- `*_network_graph.png` - Semantic network with cluster-aware layout and wiki links
- `*_tsne_scatter.png` / `*_umap_scatter.png` - 2D projections with centroid + representative labels
- `*_similarity_heatmap.png` - Pairwise cosine similarities with cluster boundary lines
- `*_dendrogram.png` - Hierarchical clustering with colored branches and threshold line
- `*_cluster_analysis.png` - Bar chart + horizontal bar (or pie for <=12 clusters) with value/% labels
- `*_clusters.json` - Cluster report with keywords, metrics, LLM names, and clustering method info

## Configuration

### Configuring for a New Wiki

Edit `setup.py` variables:

```python
WIKI_NAME = "Memory Alpha"
XML_URL = "https://s3.amazonaws.com/wikia_xml_dumps/e/en/enmemoryalpha_pages_current.xml.7z"
XML_FILENAME = "enmemoryalpha_pages_current.xml"
```

### Fandom Dump URL Pattern

```
https://s3.amazonaws.com/wikia_xml_dumps/{first_letter}/{first_two_letters}/{wikiname}_pages_current.xml.7z
```

## Data Format

### Extracted Markdown Files

Each article is saved to `data/pages/Title_With_Underscores.md`:

```markdown
# Article Title

{{sidebar|...}}

'''Article Title''' is a [[link]] to another article...
```

Only main namespace (ns=0) articles are extracted. Redirects and short articles (<200 chars) are skipped.

## GPU Configuration

- **GPU 0**: Reserved for VLM MCP tool (Qwen3-VL) -- do not use for embeddings
- **GPU 1**: Embedding model (Qwen3-Embedding-0.6B), 24GB VRAM (RTX 3090)
- Default `CUDA_VISIBLE_DEVICES=1` is set in the script
- Use `--batch-size 4` for safe VRAM usage

## LLM Cluster Naming

- Enabled via `--name-clusters` flag, requires Ollama running with `qwen3:8b`
- Ollama runs qwen3:8b on GPU locally (CUDA) -- no cloud API calls
- **Critical**: Uses `think=False` in `ollama.chat()` -- without it, Qwen3 spends all tokens on internal thinking and returns empty `content`
- **Critical**: Qwen3 sometimes outputs Chinese characters instead of English -- prompt explicitly says "in English, no Chinese characters" and code rejects non-ASCII responses via `raw.isascii()` with retry; last attempt strips non-ASCII chars
- Growing names list: clusters processed largest-first, each prompt includes all previously assigned names to prevent duplicates
- Context per cluster: TF-IDF top 30 distinctive words + 15 sample articles with title and content snippet (~200 chars each)
- Retry logic (3 attempts) handles verbose responses, duplicates, empty results, and non-English responses
- Prompt instructs model to avoid using the wiki's general topic as prefix

## CPU Safety (Critical for Large Datasets)

Running on 10K+ articles without CPU limits **will freeze the machine**. Root causes and mitigations:

| Problem | Root Cause | Fix |
|---------|-----------|-----|
| node2vec OOM | `multiprocessing` fork copies entire graph per worker | `workers=1` (no fork) |
| All cores saturated | UMAP/t-SNE/HDBSCAN default to `n_jobs=-1` | All capped at `n_jobs=2` |
| BLAS thread explosion | numpy/scipy use all cores via OpenBLAS/MKL | `OMP_NUM_THREADS=2` env vars |
| No CPU pinning | `nice` only changes priority, doesn't limit cores | `taskset -c 0-5` (6 of 14 cores) |
| O(n^2) category edges | Nested all-pairs loop over 59K articles | Group by category, cap at 500 |
| spring_layout too slow | O(n^2) per iteration on 58K nodes | 15 iterations for n>5000 |

Current safe settings in generate_graphs.py:
- node2vec: `workers=1` (walk generation), `workers=2` (Word2Vec training), `num_walks=5`
- UMAP: `n_jobs=2`
- t-SNE: `n_jobs=2`
- HDBSCAN: `core_dist_n_jobs=2`
- spring_layout: 15 iterations for graphs >5000 nodes

Full 59K-article run validated: ~90 min, peak 11.6GB RAM, no swap, system responsive throughout.

## SQLite & GPU Stability (Critical for Large Datasets)

The embedding pipeline writes 59K+ rows to SQLite and runs GPU inference for ~60 minutes. Two stability fixes are in place:

| Problem | Root Cause | Fix |
|---------|-----------|-----|
| `sqlite3.OperationalError: database or disk is full` | Per-row `conn.commit()` in delete journal mode (59K commits = 59K journal file create/delete cycles) | WAL journal mode + batched commits every ~200 embeddings |
| `CUDA error: unspecified launch failure` after ~30 min | VRAM fragmentation from sustained batch inference | `torch.cuda.empty_cache()` every ~200 embeddings |
| Lost progress on GPU crash | `sys.exit(1)` without committing pending writes | Added `conn.commit()` in CUDA error handler before exit |

SQLite settings in `init_database()`:
- `PRAGMA journal_mode=WAL` -- append-only log instead of delete journal
- `PRAGMA synchronous=NORMAL` -- safe with WAL, reduces fsync overhead

If a CUDA crash occurs, the script saves progress and exits. Re-running picks up from cache automatically.

## Performance Notes

- Default run with no `--limit` processes ALL articles (~59K for Memory Alpha)
- Embeddings are cached in SQLite (WAL mode), so re-runs only regenerate missing embeddings + graph embeddings + visualizations
- HDBSCAN with auto-scaled `min_cluster_size` adapts to dataset size: `max(5, len(articles) // 200)`
- Full 59K articles: ~90 min (CPU-safe mode), peak 11.6GB RAM, no swap
- Breakdown: node2vec ~35 min, spring_layout ~40 min, everything else ~15 min
- 10K articles: ~2 min (embeddings cached), ~5 min (fresh)

## Requirements

```bash
pip install sentence-transformers matplotlib scikit-learn networkx umap-learn mwparserfromhell node2vec gensim adjustText seaborn hdbscan
```

For LLM cluster naming: `pip install ollama` and run `ollama pull qwen3:8b`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calhounpaul)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/calhounpaul)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
