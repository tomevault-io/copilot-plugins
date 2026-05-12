## dragon-brain

> Drop this file into your project root or reference it from your Claude Code config to teach Claude how to use its persistent memory layer.

# Dragon Brain — CLAUDE.md

Drop this file into your project root or reference it from your Claude Code config to teach Claude how to use its persistent memory layer.

## The Harness

**Assertive pushback is non-negotiable. See global `~/.claude/CLAUDE.md` § The Harness — 6 guardrails against yes-man behavior. Do NOT hedge-then-agree. Say NO first, make Tabish argue his case. Devil's advocate on every decision. Name the opportunity cost.**

## What This Is

A persistent memory system for AI agents. Knowledge graph (FalkorDB) + vector search (Qdrant) + MCP server. Any MCP-compatible client can store entities, observations, and relationships — then recall them semantically across sessions. Published on PyPI as `dragon-brain`. **v1.1.0 — 100% recall@5 on LongMemEval (ICLR 2025), no LLM required.**

## Current Architecture

6-channel parallel retrieval pipeline, all channels fire on every query:
- **Dense vector** (Qdrant, BGE-M3 1024d) — semantic similarity
- **FTS5 lexical** (SQLite BM25) — keyword matches embeddings miss
- **Entity-first** (spaCy NER → FalkorDB graph) — entity → MENTIONED_IN → sessions
- **Temporal** (date parser → timeline query) — time-window filtering
- **Relational** (graph traversal) — shared entity connections
- **Associative** (spreading activation) — energy propagation through graph

Fusion: weighted RRF (k=35, PIT percentile normalization). Intent classifier sets per-channel weights (soft routing, no hard gate). Optional cross-encoder reranking (ms-marco-MiniLM, GPU/CPU auto-detect).

## Setup Verification

At the start of every session, verify the memory system is running:

```
docker ps --filter "name=claude-memory"
```

You should see 4 healthy containers: graphdb, qdrant, embeddings, dashboard.

If MCP tools (`search_memory`, `create_entity`, etc.) are not available, the server may need restarting. MCP failures are **silent** — always verify tool availability at session start.

## Updating

```bash
cd claude-memory-mcp
git pull origin master
pip install -e ".[dev]"
```

If Docker images changed: `docker compose pull && docker compose up -d`

## How to Search

### Default (Hybrid Search — Recommended)

```
search_memory(query="your question here")
```

No strategy parameter needed. The default path:
1. Runs vector similarity search (always)
2. Detects query intent (temporal, relational, associative, or semantic)
3. Enriches results with graph signals based on detected intent
4. Merges via Reciprocal Rank Fusion if graph results found entities that vector search missed

### Explicit Strategies (When You Know What You Want)

| Strategy | When to Use | Example |
|----------|-------------|---------|
| `"semantic"` | Pure meaning-based similarity | `search_memory(query="distributed systems", strategy="semantic")` |
| `"temporal"` | Time-based queries | `search_memory(query="last week's work", strategy="temporal")` |
| `"relational"` | Path/connection queries (quote entity names) | `search_memory(query="path between \"Auth\" and \"Database\"", strategy="relational")` |
| `"associative"` | Spreading activation through the graph | `search_memory(query="related to authentication", strategy="associative")` |

### Temporal Window

Temporal queries default to a 7-day lookback. Widen if needed:

```
search_memory(query="recent progress", temporal_window_days=14)
```

Use `include_meta=True` to see if there are more results beyond the window:

```
search_memory(query="recent work", include_meta=True)
```

If the response includes `"temporal_exhausted": true`, widen the window for more history.

### Understanding Results

Each result includes:

| Field | Meaning |
|-------|---------|
| `score` | Primary ranking score (cosine similarity, RRF composite, or activation composite) |
| `retrieval_strategy` | What generated this result: `"semantic"`, `"hybrid"`, `"temporal"`, `"relational"`, `"associative"` |
| `vector_score` | Raw cosine similarity from Qdrant. `null` if entity had no vector match |
| `recency_score` | 0-1 exponential decay. 1.0 = just created, 0.5 = ~7 days old |
| `activation_score` | Spreading activation energy (associative results only) |
| `path_distance` | Graph hops from query anchor (relational results only) |
| `salience_score` | Entity importance/frequency score |

**Key insight:** If `score` is 0.0, check `retrieval_strategy` — it tells you why. A temporal-only result with no vector embedding will legitimately have `score=0.0` and `vector_score=null`.

## How to Store Memories

### Entities (Things)

```
create_entity(name="Project Alpha", node_type="Entity", project_id="my-project")
```

Common node types: `Entity`, `Concept`, `Session`, `Breakthrough`, `Tool`, `Decision`, `Bottle`, `Analogy`, `Issue`, `Project`, `Procedure`, `Person`

### Observations (Facts About Things)

```
add_observation(entity_id="<uuid>", content="This project uses a microservices architecture")
```

Observations are automatically embedded and indexed for semantic search. **Adding an observation also re-embeds the parent entity** — entity vectors include observation content for richer semantic matching (not just name/type/description).

### Relationships (Connections)

```
create_relationship(
    from_entity="<uuid>",
    to_entity="<uuid>",
    relationship_type="DEPENDS_ON"
)
```

Common edge types: `RELATED_TO`, `ENABLES`, `IMPLEMENTS`, `DEPENDS_ON`, `PRECEDED_BY`, `PART_OF`, `EVOLVED_FROM`, `SUPERSEDES`

**Wiring rule:** Every entity should have at least one relationship to another entity. Entities connected only to their observations are "near-orphans" — findable via search but invisible to graph traversal.

## How to Explore the Graph

| Tool | Purpose |
|------|---------|
| `get_neighbors(entity_id, depth=1)` | Find connected entities within N hops |
| `traverse_path(from_id, to_id)` | Shortest path between two entities |
| `get_evolution(entity_id)` | Chronological history of an entity's observations |
| `find_cross_domain_patterns(entity_id)` | Non-obvious connections across domains |
| `get_hologram(query, depth=1)` | Rich subgraph visualization around a topic |

## Semantic Radar — Relationship Discovery

Discovers potential relationships by comparing vector similarity against graph distance. **Advisory only — never auto-commits edges.** Use these tools to find missing connections in the graph.

### Entity-Level Radar

```
semantic_radar(entity_id="<uuid>", limit=10, similarity_threshold=0.6)
```

For a single entity, finds semantically similar entities that are poorly connected or disconnected in the graph. Returns scored suggestions with:
- `cosine_similarity` — vector similarity score
- `graph_distance` — shortest path length (`null` = disconnected)
- `radar_score` — composite score: `cosine_sim * ln(1 + graph_distance)`. Higher = bigger gap worth bridging
- `suggested_relationship` — heuristic EdgeType (e.g., `ANALOGOUS_TO`, `BRIDGES_TO`, `ENABLES`)
- `reasoning` — human-readable explanation

Entities already directly connected (graph distance ≤ 1) are filtered out automatically.

### Batch Project Scanner

```
find_semantic_opportunities(project_id="my-project", limit=20, similarity_threshold=0.65, min_graph_distance=3)
```

Scans all entities in a project (capped at 200) to find the highest-value bridge opportunities. Deduplicates bidirectional pairs. Use this for periodic graph hygiene — "show me all missing connections."

`min_graph_distance=3` is intentionally more aggressive than entity-level radar's `≤ 1` filter — batch scanning surfaces only significant gaps.

### Weak Connection Analysis (Advanced)

After running `search_associative()`, you can pipe the activation map and vector scores into `detect_weak_connections()` (on the `ActivationEngine`) to identify:
- **Bridge opportunities** — vector-similar but graph-unreachable entities
- **Questionable edges** — graph-connected but semantically dissimilar entities

This is a standalone utility, not an MCP tool. Use it programmatically when doing deep graph analysis.

## Benchmark

Dragon Brain scores **100% recall@5** on LongMemEval (ICLR 2025), the industry-standard AI memory benchmark. 500 questions, 6 categories, no LLM required.

Full results: [benchmarks/longmemeval/RESULTS.md](benchmarks/longmemeval/RESULTS.md)

### Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `RADAR_MAX_DISTANCE_FACTOR` | `5.0` | Heuristic distance for disconnected entities (scored as `ln(1 + factor*10)`) |
| `RADAR_CONCURRENCY` | `10` | Max concurrent graph queries in batch scanner |

## How to Track Time

| Tool | Purpose |
|------|---------|
| `query_timeline(start, end)` | Entities within a time window |
| `get_temporal_neighbors(entity_id)` | Entities connected by temporal edges |
| `point_in_time_query(query, as_of)` | "What did I know as of this date?" |
| `diff_knowledge_state(as_of_start, as_of_end)` | "What changed between two dates?" — added/removed/evolved entities and relationships |
| `start_session(project_id, focus)` | Begin a tracked session |
| `end_session(session_id, summary)` | Close a session with outcomes |

### Time Diff

```
diff_knowledge_state(
    as_of_start="2026-03-01T00:00:00Z",
    as_of_end="2026-04-01T00:00:00Z",
    project_id="dragon-brain"
)
```

Shows what changed in your knowledge graph between two points — added/removed/evolved entities, new/removed relationships, and supersessions. Use for monthly reviews, sprint retrospectives, or "what did I learn last week?" Add `include_observations=True` for per-entity observation diffs (verbose).

## Health & Diagnostics

```
graph_health()          # Node/edge counts, orphans, density
system_diagnostics()    # Infrastructure status, embedding health
```

If `orphan_count > 0`, investigate before deleting — orphans may carry real data.

### Orphan Management

```
list_orphans(limit=50)   # See all orphan nodes for triage
```

### Drift Detection

```
search_stats()           # Rolling-window search behaviour stats (DRIFT-002)
```

Use `search_stats()` to monitor retrieval strategy distribution, score percentiles, and latency trends. Useful for detecting when something has drifted.

## Session Best Practices

1. **Start:** Verify containers are healthy. Run `search_memory(query="recent work")` to pick up context.
2. **During:** Log important learnings to the graph as you go. Autocompact can clear context without warning.
3. **End:** Create entities for key decisions/learnings. Update relationships. Check `graph_health()`.

## Common Pitfalls

- **MCP failures are silent.** If `search_memory` isn't available, the server may have crashed. Check Docker.
- **Don't pass `strategy="auto"`.** It's deprecated. Just omit the strategy parameter for hybrid search.
- **Observations need entities.** You can't create a free-floating observation — it must be attached to an entity.
- **Graph name is `claude_memory`**, not `dragon_brain` or anything else. If querying FalkorDB directly, use this name.
- **Subagents can't use MCP tools.** Never delegate memory operations to background agents — they don't have MCP access.

---
> Source: [iikarus/Dragon-Brain](https://github.com/iikarus/Dragon-Brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
