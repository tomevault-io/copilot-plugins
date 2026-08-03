## idea-stream

> This repo maintains a live rated arXiv stream in `arxiv.db`.

## Purpose

This repo maintains a live rated arXiv stream in `arxiv.db`.

The operating philosophy is:

1. List broadly from arXiv categories `cs.RO`, `cs.AI`, and `cs.LG`.
2. Ingest papers cheaply into a master table first.
3. Filter fast by topic relevance using only title + abstract.
4. Spend expensive web-enabled analysis only on papers that already look relevant.

The system is optimized for throughput first, then precision:

- Fetch everything from the source categories.
- Use relevance scoring as the cheap triage layer.
- Use quality scoring as the expensive enrichment layer.

## Topic Register

- `humanoid`: whole-body control, locomotion, loco-manipulation, teleoperation, sim-to-real, and deployable methods for full humanoids. Reference rubric: `rubrics/relevance_humanoid.md`.
- `intelligence`: large-scale usable representations, world models, and foundation-model style backbones that are directly valuable as general-purpose intelligence assets. Reference rubric: `rubrics/relevance_intelligence.md`.

## Architecture

The repo is built around one SQLite database, `arxiv.db`, with four tables:

- `papers`: topic-independent master paper store.
- `relevance`: per-paper, per-topic relevance scores.
- `quality`: topic-independent detailed quality annotations.
- `daemon_state`: persisted cursors and daemon bookkeeping.

Runtime is a three-stage pipeline:

- `fetch_daemon.py`: enumerates `cs.RO`, `cs.AI`, and `cs.LG` from arXiv and writes raw papers into `papers`.
- `relevance_daemon.py`: scores new papers for each registered topic using title + abstract only.
- `quality_daemon.py`: adds detailed quality annotations for papers whose max relevance already clears a threshold.

`supervisor.py` runs and restarts those three daemons.
`server.py` reads `arxiv.db` and serves the browser UI.

## Fetch Philosophy

The fetch layer should be broad and dumb, not clever.

- Do not try to pre-filter at fetch time by keywords.
- The upstream source of truth is the union of `cs.RO`, `cs.AI`, and `cs.LG`.
- Fetch into `papers` first, then let downstream scoring decide what matters.

`fetch_daemon.py` uses per-day date-windowed arXiv API queries so the system can backfill deep history without depending on large `start` offsets. ArXiv pagination becomes unreliable deep in history; day-by-day fetches avoid that failure mode.

Conventions:

- Deduplicate by bare arXiv id, without version suffix.
- Keep source data in `papers` even if it later scores low.
- Treat ingestion completeness as more important than early filtering.

## Relevance Philosophy

Relevance is the fast filter.

- Use only title + abstract.
- Do not use web tools.
- Score papers for each topic independently.
- Use the full 1-10 range instead of collapsing everything into "good" or "bad".

Relevance answers one question: "Should this paper advance to deeper inspection for this topic?"

That means:

- relevance should be cheap enough to run over the whole fetched stream.
- relevance should be stricter than broad category membership.
- relevance is allowed to be topic-specific and asymmetric across topics.

## Quality Philosophy

Quality is the expensive annotation layer.

- Only run after relevance has already filtered the stream.
- Quality is topic-independent.
- Use web research to determine institution, venue, citations, impact, and overall paper quality.

Current composite quality is:

`0.4 * institution + 0.3 * venue + 0.2 * impact + 0.1 * quality_score`

This stage exists to enrich already-promising papers, not to rescue irrelevant ones.

## Runtime Rules

- Main database: `arxiv.db`.
- Live run id: `live`.
- Keep SQLite in WAL mode for the concurrent daemon workload.
- Each daemon should issue at most one agent call at a time.
- The intended parallelism is one fetch worker, one relevance worker, and one quality worker.

## Run Commands

```bash
python3 supervisor.py
python3 server.py
```

Supervisor defaults: relevance uses `claude-code`; quality uses `opencode`.

Typical standalone use:

```bash
python3 fetch_daemon.py --max-pages 1000
python3 relevance_daemon.py --cycles 1
python3 quality_daemon.py --cycles 1
```

## Important Files

- `lib.py`: schema, DB helpers, arXiv id normalization, and agent backend invocation.
- `fetch_daemon.py`: broad category ingestion from arXiv.
- `relevance_score.py`: one-shot relevance scorer used by the daemon.
- `quality_score.py`: one-shot quality annotator used by the daemon.
- `rubrics/relevance_humanoid.md`: humanoid relevance rubric.
- `rubrics/relevance_intelligence.md`: intelligence relevance rubric.
- `rubrics/quality.md`: quality rubric.
- `schemas/relevance.schema.json`: structured relevance output contract.
- `schemas/quality.schema.json`: structured quality output contract.

## What To Preserve

When modifying the system, preserve these invariants:

- Fetch stays broad across `cs.RO`, `cs.AI`, and `cs.LG`.
- Relevance stays cheap and fast.
- Quality stays detailed and gated behind relevance.
- `papers` is the master store; topic logic belongs in `relevance`, not fetch.
- Quality remains topic-independent so one annotation can serve multiple topic views.

## What To Avoid

- Do not reintroduce keyword-driven fetch as the primary discovery mechanism.
- Do not push expensive web research into the first-pass relevance stage.
- Do not couple quality scoring to a single topic.
- Do not add back operator state files when the database already holds the needed runtime state.
- Do not introduce another primary database beside `arxiv.db`.

---
> Source: [5eqn/idea-stream](https://github.com/5eqn/idea-stream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
