## knowledge-base

> A personal knowledge engine that transforms passive reading into structured understanding of the AI field. Ingests sources (papers, articles, videos, podcasts), extracts claims with evidence, discovers connections, generates ideas through multi-agent debate, and maintains a living model of the current state of AI — tracking capabilities, limitations, bottlenecks, breakthroughs, and anticipations across thematic areas.

# Knowledge Base — Agent Instructions

A personal knowledge engine that transforms passive reading into structured understanding of the AI field. Ingests sources (papers, articles, videos, podcasts), extracts claims with evidence, discovers connections, generates ideas through multi-agent debate, and maintains a living model of the current state of AI — tracking capabilities, limitations, bottlenecks, breakthroughs, and anticipations across thematic areas.

## Architecture

Local-first. PostgreSQL + pgvector + FTS as the single database. Markdown memory files (human-auditable). SQLite job queue. Telegram interface via long polling. File-based library (`library/{source_id}/`). Gateway → Adapters (Telegram, CLI, Heartbeat) → Skills Registry → Agent Executor → Storage. No cloud dependencies.

## The Three Layers

- **Knowledge graph** — what sources say. Claims, concepts, edges, evidence. Every claim links back to a verbatim snippet.
- **Landscape model** — what is true about AI right now. Themes (DAG), capabilities (with maturity), limitations (with type and trajectory), bottlenecks (with resolution horizon), breakthroughs (with implication cascades), anticipations (trackable predictions), cross-theme implications.
- **Belief system** — what you think about what is true. Tracked positions with confidence, evidence for/against, landscape links, history. The system surfaces conflicts and challenges stale positions.

## Core Principles

- **Evidence-traced.** Every claim links back to a verbatim snippet. Every edge has evidence from both sides.
- **Temporally aware.** The landscape model is a trajectory, not a snapshot. State summaries describe evolution, momentum, and stalling.
- **Human-enriched.** Automated extraction detects what's explicit and implicit. Highest-value insights come from the user via `/enrich`, `/implications`, and `/challenge`.
- **Limitations are the most valuable signal.** The system extracts implicit limitations alongside explicit ones, classifies them by type and severity, and links them to bottlenecks.

## Generated Wiki Indexes

Do not hand-edit generated wiki index rows in `wiki/index.md` or `wiki/*/index.md`. Index display rows are derived artifacts built from canonical page frontmatter/body. If an index row is wrong, fix the underlying page or metadata, then rebuild the index with `retrieval.wiki_index.rebuild_index()` or the relevant narrower rebuild helper. If an index shows mojibake, regenerate it from clean pages; do not copy terminal output back into Markdown.

## How to Invoke Skills

All skills can be invoked via the test harness:

```bash
python scripts/test_skill_harness.py <skill_name> "<slash_command>"
```

**Requirements:** PostgreSQL must be running with the knowledge base schema. Environment variables in `.env` must be configured (see `.env.example`).

**What to expect:** The harness imports the handler, constructs a job object, and executes it. Output is printed to stdout. Some skills stream progress updates. Long-running skills (save, reflect deep, synthesis) may take several minutes.

Alternatively, read the skill's prompt file at `skills/prompts/<skill_name>.md` and follow the instructions directly using your own tools to query the database.

---

## Skills Reference

### Ingestion & Management

#### save
Ingest a URL into the library with full extraction pipeline (theme classification, claim extraction, landscape signals, cross-theme implications, anticipation matching, belief checking).

**Usage:** `/save <url> [time_ranges]`
**Examples:**
- `/save https://arxiv.org/abs/2401.12345`
- `/save https://www.youtube.com/watch?v=abc123 8:55-27:04, 2:34:58-2:46:17`

**Invoke:** `python scripts/test_skill_harness.py save "/save <url>"`
**Handler:** `gateway/save_handler.py` (direct)

---

#### summarise
Preview a source summary before deciding to fully ingest. Fetches URL, classifies themes, generates summary. Does NOT run claim extraction or landscape pipeline. Stages result for 24h; follow up with `/save_confirmed` to complete ingestion.

**Usage:** `/summarise <url> [time_ranges]`
**Examples:**
- `/summarise https://arxiv.org/abs/2401.12345`
- `/summarise https://www.youtube.com/watch?v=abc123 8:55-27:04`

**Invoke:** `python scripts/test_skill_harness.py summarise "/summarise <url>"`
**Handler:** `gateway/summarise_handler.py` (direct)

---

#### delete
Remove a source and all derived artifacts. Cascading DB cleanup with no LLM involvement — removes claims, concepts, edges, themes, capabilities, limitations, bottlenecks, implications, anticipations, breakthroughs, beliefs evidence, and the `library/{source_id}/` directory.

**Usage:** `/delete <source_id_or_url>`
**Examples:**
- `/delete 01HQ3XMVBN7K9P2R4S6T8W0Y1Z`
- `/delete https://arxiv.org/abs/2401.12345`

**Invoke:** `python scripts/test_skill_harness.py delete "/delete <source_id>"`
**Handler:** `gateway/delete_handler.py` (direct)

---

### Knowledge Exploration

#### ask
Natural language QA with citations. Routes internally: structured entity queries (pure DB lookup for bottlenecks, anticipations, capabilities, limitations, breakthroughs, beliefs, implications), complex questions (hybrid retrieval + LLM synthesis), and report follow-ups.

**Usage:** `/ask <question>`
**Examples:**
- `/ask What are the main limitations of AI agents?`
- `/ask show bottlenecks for robotics`

**Invoke:** `python scripts/test_skill_harness.py ask "/ask <question>"`
**Handler:** `gateway/ask_handler.py` (direct)

---

#### map
Progressive 4-level graph exploration: direct edges → shared concepts → thematic proximity → cross-theme implications. Stops when sufficient connections are found.

**Usage:** `/map <concept_or_source_id>`
**Examples:**
- `/map world models`
- `/map transformer architecture`
- `/map 01HQ3X`

**Invoke:** `python scripts/test_skill_harness.py map "/map <concept>"`
**Handler:** `skills/prompts/map.md` (LLM-dispatched)

---

#### path
Find and explain the connection path between two sources in the knowledge graph.

**Usage:** `/path <source_a> <source_b>`
**Examples:**
- `/path 01HQ3X 01HRA2`
- `/path "scaling laws" "emergent abilities"`

**Invoke:** `python scripts/test_skill_harness.py path "/path <source_a> <source_b>"`
**Handler:** `gateway/path_handler.py` (direct)

---

#### reflect
Generate connections and ideas from a source. Three modes:

| Mode | Input | Description |
|---|---|---|
| simple | `/reflect <source_id>` | Graph retrieval + hybrid search + LLM synthesis |
| deep | `/reflect <source_id> deep` | Full tournament: generate → novelty gate → critique → debate → evolve → rank |
| topic | `/reflect topic <text>` | Cross-source topic reflection with multi-lens analysis |

**Invoke:** `python scripts/test_skill_harness.py reflect "/reflect <source_id> [deep]"`
**Handler:** `gateway/reflect_handler.py` (direct)

---

#### synthesis
Deep cross-source synthesis on a topic or merge analysis across specific sources.

**Usage:**
- `/synthesis <topic>` — topic-mode synthesis
- `/synthesis <source_id> <source_id> [...]` — merge specific sources

**Invoke:** `python scripts/test_skill_harness.py synthesis "/synthesis <topic>"`
**Handler:** `skills/prompts/synthesis.md` (LLM-dispatched with prefetch)

> **Important:** See [Special Cases — Synthesis Prefetch](#synthesis-prefetch) below.

---

### Landscape Model

#### landscape
View the current AI landscape state. Pure DB query, no LLM.

**Usage:** `/landscape [theme_name]`
**Examples:**
- `/landscape` — overview of all themes by velocity
- `/landscape robotics` — detailed view with capabilities, limitations, bottlenecks, breakthroughs, anticipations

**Invoke:** `python scripts/test_skill_harness.py landscape "/landscape [theme]"`
**Handler:** `gateway/landscape_handler.py` (direct)

---

#### bottlenecks
Ranked bottleneck analysis by leverage (impact x proximity to resolution). Surfaces cascade potential, unaddressed bottlenecks, and horizon shifts.

**Usage:** `/bottlenecks [theme]`
**Examples:**
- `/bottlenecks` — all bottlenecks ranked by leverage
- `/bottlenecks robotics` — bottlenecks for robotics theme

**Invoke:** `python scripts/test_skill_harness.py bottlenecks "/bottlenecks [theme]"`
**Handler:** `skills/prompts/bottlenecks.md` (LLM-dispatched)

---

#### anticipate
Generate, review, and calibrate predictions about what comes next.

**Usage:** `/anticipate [subcommand] [args]`
**Invoke:** `python scripts/test_skill_harness.py anticipate "/anticipate [subcommand] [args]"`
**Handler:** `gateway/anticipate_handler.py` (direct)

**Subcommands:**

| Subcommand | Input | Description |
|---|---|---|
| (default) | `/anticipate` | Overview of anticipations needing attention |
| review | `/anticipate review [theme]` | Review against accumulated evidence |
| generate | `/anticipate generate <theme>` | Generate 3-7 predictions from landscape context |
| evolve | `/anticipate evolve <theme>` | Retire stale predictions, regenerate |
| calibration | `/anticipate calibration` | Brier scores, calibration by confidence bucket |
| confirm | `/anticipate confirm <id>` | Mark confirmed, log to landscape history |
| invalidate | `/anticipate invalidate <id>` | Mark invalidated, log to landscape history |

---

#### changelog
View what changed in the landscape model over time. Pure DB query over `landscape_history`, no LLM.

**Usage:** `/changelog [theme] [Nd]`
**Examples:**
- `/changelog` — all changes in the last 30 days
- `/changelog robotics 14d` — robotics changes in the last 14 days
- `/changelog 7d` — all changes in the last 7 days

**Invoke:** `python scripts/test_skill_harness.py changelog "/changelog [theme] [Nd]"`
**Handler:** `gateway/changelog_handler.py` (direct)

---

#### gaps
8-part coverage gap analysis: over-optimistic coverage, blind spot bottlenecks, untested predictions, incomplete capabilities, high-velocity low-coverage themes, unlinked themes, validation backlog, belief-driven gaps.

**Usage:** `/gaps [theme]`
**Examples:**
- `/gaps` — full gap analysis across all themes
- `/gaps robotics` — gaps specific to robotics

**Invoke:** `python scripts/test_skill_harness.py gaps "/gaps [theme]"`
**Handler:** `skills/prompts/gaps.md` (LLM-dispatched)

---

#### themes
Manage the theme taxonomy that organises the landscape model.

**Usage:** `/themes [subcommand] [args]`
**Invoke:** `python scripts/test_skill_harness.py themes "/themes [subcommand] [args]"`
**Handler:** `skills/prompts/themes.md` (LLM-dispatched)

**Subcommands:**

| Subcommand | Input | Description |
|---|---|---|
| (default) | `/themes` | List all themes with source counts |
| approve | `/themes approve <id>` | Approve a proposed theme |
| reject | `/themes reject <id>` | Reject a proposed theme |
| hierarchy | `/themes hierarchy` | View the theme DAG structure |
| edges | `/themes edges` | View theme-to-theme edges |
| approve-edge | `/themes approve-edge <id>` | Approve a proposed theme edge |
| reject-edge | `/themes reject-edge <id>` | Reject a proposed theme edge |
| evolve | `/themes evolve` | Trigger theme evolution analysis |
| review-evolution | `/themes review-evolution` | Review pending evolution proposals |

---

### Human Enrichment

#### enrich
Correct, supplement, or reinterpret source extractions. All updates persisted with `attribution='user_enrichment'`.

**Usage:** `/enrich <source_id> [text_or_subcommand]`
**Invoke:** `python scripts/test_skill_harness.py enrich "/enrich <source_id> [text]"`
**Handler:** `gateway/enrich_handler.py` (direct)

**Modes:**

| Mode | Input | Description |
|---|---|---|
| inspect | `/enrich <source_id>` | Show current extractions, prompt for input |
| validate | `/enrich <source_id> validate` | List unvalidated implicit limitations |
| validate specific | `/enrich <source_id> validate <id> yes\|no` | Validate/reject specific limitation |
| validate batch | `/enrich <source_id> validate all yes\|no` | Batch validate all |
| free-form | `/enrich <source_id> <text>` | LLM parses into structured updates |

---

#### challenge
Challenge landscape entities or beliefs. LLM examines the entity against all available evidence, looking for weaknesses, missing context, or counter-evidence.

**Usage:**
- `/challenge <entity_type> <entity_id>` — challenge a landscape entity
- `/challenge belief <belief_id> steelman` — strongest counter-case against own belief

**Entity types:** `capability`, `limitation`, `bottleneck`, `breakthrough`, `anticipation`, `belief`

**Examples:**
- `/challenge bottleneck 42`
- `/challenge belief 7 steelman`

**Invoke:** `python scripts/test_skill_harness.py challenge "/challenge <entity_type> <entity_id>"`
**Handler:** `gateway/challenge_handler.py` (direct)

---

#### beliefs
Manage tracked beliefs with confidence, evidence, and staleness tracking.

**Usage:** `/beliefs [subcommand] [args]`
**Invoke:** `python scripts/test_skill_harness.py beliefs "/beliefs [subcommand] [args]"`
**Handler:** `gateway/beliefs_handler.py` (direct)

**Subcommands:**

| Subcommand | Input | Description |
|---|---|---|
| (default) | `/beliefs` | List all active beliefs |
| (topic) | `/beliefs <topic>` | List beliefs filtered by topic |
| add | `/beliefs add <statement>` | LLM classifies type/theme/confidence, dedup check |
| update | `/beliefs update <id> <confidence> [reason]` | Update confidence (0.0-1.0) |
| review | `/beliefs review` | Review stale beliefs (>30 days) against recent claims |
| synthesis | `/beliefs synthesis <topic>` | Synthesize beliefs into narrative |
| suggest | `/beliefs suggest [topic]` | Find claim clusters, suggest new beliefs |

---

#### implications
Deep cross-theme impact analysis from a source, with optional user thesis.

**Usage:**
- `/implications <source_id>` — automated 4-lens parallel analysis
- `/implications <source_id> <thesis>` — validate a specific thesis about what the source implies

**Examples:**
- `/implications 01HQ3X`
- `/implications 01HQ3X This paper's findings on world models could reshape robotics sim-to-real transfer`

**Invoke:** `python scripts/test_skill_harness.py implications "/implications <source_id> [thesis]"`
**Handler:** `gateway/implications_handler.py` (direct)

---

### Analysis & Planning

#### contradictions
Find conflicting claims across sources via embedding similarity, `source_edges` relationship types, and `cross_theme_implications` for theme-level tensions. Analyses nature, severity, and implications of each contradiction.

**Usage:** `/contradictions [topic]`
**Examples:**
- `/contradictions` — all contradictions
- `/contradictions scaling` — contradictions related to scaling

**Invoke:** `python scripts/test_skill_harness.py contradictions "/contradictions [topic]"`
**Handler:** `skills/prompts/contradictions.md` (LLM-dispatched)

---

#### ideas
Browse, rate, develop, and track ideas generated from `/reflect` tournaments.

**Usage:** `/ideas [subcommand] [args]`
**Invoke:** `python scripts/test_skill_harness.py ideas "/ideas [subcommand] [args]"`
**Handler:** `gateway/ideas_handler.py` (direct)

**Subcommands:**

| Subcommand | Input | Description |
|---|---|---|
| (default) | `/ideas` | List ideas sorted by score (limit 30) |
| (filter) | `/ideas <text_or_status>` | Filter by status or text search |
| view | `/ideas view <id>` | Full detail: scores, grounding, notes |
| rate | `/ideas rate <id> <1-5>` | Set user rating |
| note | `/ideas note <id> <text>` | Append to user notes |
| develop | `/ideas develop <id>` | LLM expands: grounding, tests, risks, next steps |
| status | `/ideas status <id> <status>` | Change status (new/developing/tested/confirmed/abandoned) |

---

#### next
Prioritised reading recommendations based on coverage gap analysis. Analyses blind spot bottlenecks, validation backlog, belief coverage gaps, untested anticipations, and low-coverage themes. Persists results to the `reading_queue` table.

**Usage:** `/next [N] [theme]`
**Examples:**
- `/next` — 5 recommendations across all themes
- `/next 10 robotics` — 10 recommendations focused on robotics

**Invoke:** `python scripts/test_skill_harness.py next "/next [N] [theme]"`
**Handler:** `gateway/next_handler.py` (direct)

---

## Special Cases

### Synthesis Prefetch

The `/synthesis` skill requires context prefetch that the test harness does **not** handle automatically. The harness will work but produce less comprehensive results.

**For full results, use direct Python invocation:**

```bash
python -c "
import sys; sys.path.insert(0, '.')
from dotenv import load_dotenv; load_dotenv()
from reading_app.db import ensure_pool; ensure_pool()
from retrieval.topic_synthesis import gather_synthesis_context, gather_merge_context
# Topic mode:
ctx = gather_synthesis_context('scaling laws')
# Merge mode:
# ctx = gather_merge_context(['01HQ3X', '01HRA2'])
print(ctx)
"
```

Then pass the context to your LLM along with the prompt at `skills/prompts/synthesis.md`.

**Or use Codex directly:** Read `skills/prompts/synthesis.md` and follow the instructions — Codex's tools can query the database and gather context natively.

---

## Common Workflows

### Ingest and explore a new source
```
/save https://arxiv.org/abs/2401.12345
/landscape                                  # see updated landscape
/reflect 01HQ3X                             # generate connections
/implications 01HQ3X                        # cross-theme impact
```

### Investigate a topic in depth
```
/ask What are the current bottlenecks for autonomous agents?
/bottlenecks autonomous_agents
/synthesis autonomous agents
/contradictions autonomous agents
```

### Maintain the knowledge model
```
/gaps                                       # find coverage holes
/next 5                                     # get reading recommendations
/beliefs review                             # review stale beliefs
/anticipate review                          # check predictions against evidence
/changelog 7d                               # recent landscape changes
```

---

## Detailed Documentation

Each skill's executable prompt lives at `skills/prompts/<skill_name>.md` with execution contracts, edge cases, and implementation details.

---
> Source: [sehyungp92/knowledge_base](https://github.com/sehyungp92/knowledge_base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
