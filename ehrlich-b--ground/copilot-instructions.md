## ground

> Ground is a source-anchored encyclopedia of weighted facts. Every claim traces back to verbatim quotes from cited sources. Sources carry credibility scores, exposed as free parameters via lenses — users can adjust source trust and re-render the entire knowledge graph in milliseconds. Agents are research workers (search/extract/audit), not voters. The mechanical containment check (`strings.Contains(source.body, citation.verbatim_quote)`) is the wall against LLM hallucination. See DESIGN.md for the full v2 specification.

# CLAUDE.md — ground

## Project Overview

Ground is a source-anchored encyclopedia of weighted facts. Every claim traces back to verbatim quotes from cited sources. Sources carry credibility scores, exposed as free parameters via lenses — users can adjust source trust and re-render the entire knowledge graph in milliseconds. Agents are research workers (search/extract/audit), not voters. The mechanical containment check (`strings.Contains(source.body, citation.verbatim_quote)`) is the wall against LLM hallucination. See DESIGN.md for the full v2 specification.

The v1 design ("12 LLM personalities argue, EigenTrust extracts truth") is archived in git history under tag `v1-final`. Do not write new code against v1 concepts (assertions, reviews, support/contest stances, helpfulness ratings, contest quotas).

## Style Guide

This project follows the same conventions as ~/repos/wingthing:

- **Single binary Go + SQLite**. No Docker, no microservices, no ORM.
- **modernc.org/sqlite** (pure Go, no CGO). WAL mode + foreign keys enabled on open.
- **Embedded migrations** via `//go:embed migrations/*.sql`. Tracked in `schema_migrations` table. Auto-run on DB open.
- **Cobra CLI** in `cmd/ground/main.go`. Version injected via ldflags.
- **Go 1.22+ http.ServeMux** routing (`"GET /topic/{slug}"`). No framework.
- **Server-rendered HTML** via Go templates. No React, no npm, no node_modules. D3.js is the only JS dependency (for graph viz).
- **REST API** under `/api/` with JWT auth. Web UI and API served on the same port from the same binary.
- **stdlib `log`** only. No logging library.
- **`fmt.Errorf("context: %w", err)`** for all error wrapping. No panics.
- **No ORM**. Raw `sql.Query`/`QueryRow` with `Scan`. Helper functions for scanning rows.
- **Pointer fields** for nullable columns (`*string`, `*time.Time`).
- **Environment variables** for secrets and infrastructure config. No config file for MVP.

## Project Structure

```
ground/
├── CLAUDE.md
├── DESIGN.md
├── SKILLS.md          Bot developer / agent integration guide
├── TOPICS.md          Seed topic map with dependency structure
├── FACTS.md           Axiomatic nodes (adjudicated at seed time)
├── TODO.md
├── Makefile
├── README.md
├── go.mod / go.sum
├── cmd/ground/main.go       Cobra root + subcommands (server + client mode)
├── internal/
│   ├── db/                  SQLite open, migrations, query methods
│   │   └── migrations/      Numbered .sql files (001_init.sql, 002_v2_schema.sql, ...)
│   ├── model/               Data types (Agent, Topic, Claim, Source, Citation, Audit, Lens, Dependency, Epoch)
│   ├── sources/             Source fetching, body blob storage, content-hash dedup, citation graph extraction
│   ├── lens/                Per-request lens render: sparse merge, linear groundedness pass, topo DAG
│   ├── agent/               Seed orchestration — registers agents, launches claude -p
│   ├── client/              HTTP client for remote Ground instances (CLI client mode)
│   ├── engine/              Per-epoch baseline: source credibility, agent reliability, claim groundedness
│   ├── embed/               Embedding generation (OpenAI), cosine similarity, duplicate detection
│   ├── api/                 REST API handlers, JWT auth middleware, rate limiting
│   └── web/                 HTML handlers, template rendering, lens-aware views
├── prompts/                 Seed agent personality files (search/extraction strategy biases — NOT epistemic stances)
├── tasks/                   Seed round task descriptions (v2-search.md, v2-extract.md, v2-audit.md, v2-deps.md)
├── templates/               Go HTML templates
├── static/                  CSS, D3.js
└── ground.db                (gitignored)
```

## Key Concepts

- **Sources** are first-class. Every source has a fetched, cached body (content-addressed by sha256), metadata, tags, and a credibility score.
- **Citations** link a claim to a source via a verbatim quote that must literally appear in the cached body. The mechanical containment check is the wall against LLM hallucination — failing it returns 400 before any LLM judgment runs.
- **Audits** are agents verifying other agents' citations. Two stages: server-automatic mechanical re-check, then LLM-driven semantic verdict (`confirm`, `misquote`, `out_of_context`, `weak`, `broken_link`). Auditors cannot audit their own citations.
- **Agents** are research workers. Roles: `extractor`, `auditor`, `both`, `observer`, `admin`. Scored on **reliability** (audit-weighted citation pass rate) and **productivity** (capped throughput).
- **Claims** are atomic propositions with citations. Intrinsic groundedness is the credibility-weighted balance of supporting / contradicting / qualifying citations — linear in source credibility (this is what makes lens render cheap). Effective groundedness flows through a dependency DAG.
- **Adjudicated claims** are pinned by admin (FACTS.md), and now require ≥1 valid citation to a Tier-1 anchored source.
- **Source credibility** has three layers: anchor priors (admin-curated, public), computed baseline (per-epoch from anchors + citation graph + audit aggregate), lens overrides (per-user sparse map).
- **Lenses** are first-class saveable, forkable, shareable views. Per-source overrides and per-tag multipliers. Every read endpoint accepts `?lens=slug`. Lenses move source credibility but **never** agent reliability, citation existence, or anchor priors.
- **Dependencies** form a DAG. Effective groundedness flows through it via topological order. Cycle detection at insertion.
- **Personality prompts** (12 v1 files) are demoted to search/extraction strategies. They bias *what sources an agent looks for*, not *what stance they take*. Forced contest quotas are deleted.
- **REST API** with JWT auth is first-class. Bot contributors register, then either audit (lowest-risk reliability building) or extract.
- **CLI dual mode** — server commands (serve, compute, anchor, source, bootstrap) and client commands (login, cite, audit, claim, depend, lens, show, explore). Same binary.

## Build & Run

```
make build              # produces ./ground binary
./ground serve          # start web server + API on :8080
./ground compute        # run one epoch (source credibility, agent reliability, claim groundedness)

# Bootstrapping:
./ground anchor add <url> --tier 1 --credibility 0.92 --reasoning "..."
./ground bootstrap-anchors anchors.yaml
./ground bootstrap-axioms FACTS.md
./ground source ingest <url>
./ground source refresh --all

# Client mode (after ground login):
./ground login https://ground.ehrlich.dev
./ground cite <claim-id> --source <url> --quote "..." --polarity supports --reasoning "..."
./ground audit <citation-id> --semantic confirm --reasoning "..."
./ground lens new --slug primary-only
./ground lens set primary-only --tag preprint --multiplier 0
./ground leaderboard
```

## Environment Variables

```
GROUND_JWT_SECRET       Required (server mode). JWT signing key.
OPENAI_API_KEY          Required for embeddings (text-embedding-3-small).
GROUND_PORT             Server port (default 8080).
GROUND_URL              Remote server URL (client mode, also set by ground login).
```

## Testing

```
make test               # go test ./...
```

## Important Implementation Notes

### The mechanical wall (most important rule)
- Every citation submission runs `strings.Contains(source.body, citation.verbatim_quote)` BEFORE any LLM judgment. Failing this returns 400. No exceptions.
- Re-run the same check during audit (drift detection); if a re-fetched source body no longer contains the quote, mechanical=fail and the citation is hard-rejected regardless of any prior semantic verdict.
- Auditors cannot audit their own citations.

### Algorithm
- Per-epoch baseline:
  - Source credibility EigenTrust over the source-source citation graph, blended with anchor priors and audit aggregates.
  - Agent reliability EigenTrust over audit verdicts (uphold/reject), with mechanical-fail as hard reject.
  - Claim intrinsic groundedness is **linear** in source credibility — this is what makes lens render cheap. Polarity coefficients: `+1 supports, -alpha contradicts, +beta qualifies`.
  - Effective groundedness in topological order over the dep DAG.
  - Adjudicated claims pinned and skipped.
- Per-request lens render:
  - Sparse merge of lens overrides into baseline credibility map.
  - Linear pass over claims, topo pass over deps.
  - Cache by `(lens_id, epoch_id)`.
- Lenses move source credibility ONLY. Agent reliability is computed off baseline and never lens-rendered.

### Data Integrity
- Citations cannot be deleted; if invalidated by drift or audit, status is set, but the row stays.
- Audits cannot be deleted.
- Claims cannot be deleted. Adjudicated claims cannot be moved by the algorithm.
- Dependencies must not create cycles (validate before insert).
- Duplicate claims rejected if cosine similarity > 0.95 with existing.
- Source bodies are immutable per content_hash. New body on refetch = new hash = drift event recorded.

### API
- JWT tokens expire after 90 days. Rotation via POST /api/agents/token.
- Rate limits: 100 citations/day, 500 audits/day, 50 claims/day, 100 source candidates/day, 10 rps burst.
- All responses: `{data, meta}` or `{error: {code, message, details}}`.
- Discovery endpoints (leaderboard, contested, frontier, graph) are unauthenticated and lens-aware via `?lens=slug`.
- Topic creation restricted to admin and seed agents.
- Anchor management restricted to admin role.

### Content
- Claims belong to topics by embedding proximity, not foreign key. No `topic_id` on claims.
- Source bodies stored content-addressed under `~/.ground/blobs/{sha256}`; database holds metadata only.
- Citation graph between sources is auto-extracted where feasible (DOI references, HTML hyperlinks, PDF bibliographies).
- Anchor list is small (~100), public, version-controlled in `anchors.yaml`. Adding/removing an anchor is a deliberate, reviewable change.

### What v1 concepts are dead
- `assertions` table (support/contest/refine + confidence) — deprecated, will be dropped in `003_drop_v1.sql`. Replaced by `citations` (claim ↔ source with verbatim quote, polarity).
- `reviews` table (helpfulness ratings) — deprecated. Replaced by `audits` (objective verdicts on citation correctness).
- `agents.accuracy`, `agents.contribution`, `agents.weight` — deprecated. Replaced by `reliability` and `productivity`.
- "Helpfulness" as a score concept — gone. Citations either survive audit or they don't.
- Forced contest quotas in seed scripts — deleted entirely.

---
> Source: [ehrlich-b/ground](https://github.com/ehrlich-b/ground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
