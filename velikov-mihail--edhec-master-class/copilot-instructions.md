## edhec-master-class

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A 6-hour master class for EDHEC (Department of Finance, Economics, and Data Science), May 6, 2026: "Using AI for Research: From Chatbots to Autonomous Pipelines." Two artifacts coexist here:

1. **Beamer slide decks** at the repo root.
2. **Six self-contained live-demo workspaces** (`demo1-skill/`, `demo2-voice-skill/`, `demo3-fred-agent/`, `demo4-referee-skill/`, `demo5-strategic-revision/`, `demo6-replication-pipeline/`) that the audience watches Claude Code drive in real time. Demos 2 and 5 are skip-if-behind buffers; the other four are the spine.

The decks and demos are tightly coupled: slides cite the demo skills/agents by name (e.g., `/edhec-summary-stats`, `/referee-report`, the four pipeline stages). Renaming a skill or restructuring a demo's `.claude/` tree means slide text needs to follow.

## Slide decks

Two `.tex` files, both standalone Beamer documents using the same preamble (no shared `\input`):

- `morning.tex` — concept session. Sections cover Voices from the Field → Evolution of AI → How an Agent Works → Choosing Your Tools → Working with Claude Code → Practical Prompting & Pitfalls → Skills → Subagents → Hooks.
- `afternoon.tex` — demo-led session. Walks through Demos 1–6 (Demos 2 and 5 are skip-if-behind buffers), then CLAUDE.md programming principles, the Auto-AI-Research pipeline, and Implications.

Both share the same Beamer preamble: aspect ratio 16:9, custom color palette (`darkblue`/`accentorange`), TikZ styles for `auditbase`/`agentbase`/`phasebase`, and `tcolorbox` styles `keyinsight`/`warningbox`/`bigquote`/`codeblock`. **Do not introduce a new color or tcolorbox style without checking it doesn't collide with existing aliases** (`navy`, `gold`, `coral`, `sage`, etc. are all `\colorlet` aliases for the four-color palette).

### Building slides

```bash
pdflatex morning.tex   # or: latexmk -pdf morning.tex
pdflatex afternoon.tex
```

LaTeX aux files (`*.aux`, `*.log`, `*.nav`, `*.snm`, `*.toc`, `*.synctex.gz`, `*.fdb_latexmk`, `*.fls`, `*.out`, `*.vrb`) are gitignored. Generated PDFs (`morning.pdf`, `afternoon.pdf`) are committed because they're the deliverable.

## Demo workspaces

Each `demoN-*/` folder is a **self-contained Claude Code workspace**. They are not packaged as a monorepo — open each one in its own Claude Code session when developing or rehearsing.

| Folder | Purpose | Time slot | Key surface |
| --- | --- | --- | --- |
| `demo1-skill/` | Build a single skill (`edhec-summary-stats`) live, from a YAML skeleton. | ~15 min | `.claude/skills/edhec-summary-stats/SKILL.md`. |
| `demo2-voice-skill/` | **Skip-if-behind buffer.** Single skill (`voice-extractor`) reads a corpus of your own published papers (PDFs) and produces a `VOICE.md` style guide. Verify phase greps each "recurring phrase" claim against the source PDFs. | ~15 min (only if on schedule) | `.claude/skills/voice-extractor/SKILL.md`. Drop your own paper PDFs into `sample-papers/`. |
| `demo3-fred-agent/` | Hand-write a Think→Act→Observe agent against the FRED API using the Anthropic SDK. | ~20 min | `agent_skeleton.py` (two TODO blocks filled live) + `agent_reference.py` (full working version). |
| `demo4-referee-skill/` | One orchestrator skill spawns three reviewer subagents in parallel, then consolidates. | ~25 min | `.claude/skills/referee-report/SKILL.md` + `.claude/agents/{empirical-econ-reviewer,theory-reviewer,methods-critic}.md`. Drop a working-paper PDF into `sample-paper/`. |
| `demo5-strategic-revision/` | **Skip-if-behind buffer.** Tour and run Jukka Sihvonen's `strategic-revision` skill (Aalto; featured by Goldsmith-Pinkham on Markus' Academy): manuscript + editor letter + reviewer reports → DAG-validated revision plan with execution blocks. Verify gate is *computational* (NetworkX acyclicity check). | ~15 min (only if on schedule) | `.claude/skills/strategic-revision/SKILL.md`. Drop manuscript PDF into `manuscript/`, reviewer comments into `reviews/`. |
| `demo6-replication-pipeline/` | **Capstone.** Mini autonomous CAPM-replication pipeline with state-machine + JSON schema validation + verifier subagent. | ~35 min | See architecture below. |

### Demo 6 architecture (load-bearing)

Demo 6 is the most intricate and the highest-risk live demo. Its own `CLAUDE.md` at `demo6-replication-pipeline/CLAUDE.md` is authoritative; key invariants to preserve when editing:

- **State is single-source-of-truth.** `state/state.json` drives stage routing (`init → fetched → regressed → tabulated → summarized → done`, plus `halted`). `schema/state.v1.json` is the JSON Schema; every write must validate.
- **Per-stage verification.** After each skill writes state, it runs `python scripts/verify_state.py` (jsonschema validator, exits 0/2/3) AND spawns the `verifier` subagent (`.claude/agents/verifier.md`) to cross-check artifacts against state with 1e-4 tolerance. Verifier replies exactly `OK` or `DISCREPANCY: …`; the orchestrator parses mechanically.
- **Mechanical termination.** Stop on `iteration >= max_iterations` OR `delta < delta_threshold`. Never on "I think we're done."
- **WRDS-only fetch.** `scripts/fetch_wrds.py` is the sole data source: requires `WRDS_USERNAME` env var so libpq matches `pgpass.conf`; exits non-zero on any failure (the orchestrator then halts the pipeline — there is no fallback). CSV schema: `date, permno, ticker, ret, mkt_rf, rf`.
- **Skills delegate decisions to the LLM, not hardcoded params.** `fetch-data` asks the orchestrator to *pick* 8 large-cap tickers across ≥3 sectors and write the rationale into `state.last_action.note` — that note is the claim the verifier checks the CSV against. Preserve this design pattern in any new skill.
- **One orchestrator, one session.** The Claude Code session IS the orchestrator. No external scheduler.

### Demo 6 setup commands

```bash
pip install jsonschema statsmodels pandas wrds python-dotenv
cp .env.example .env                    # then fill in WRDS_USERNAME
# (libpq reads the matching password from pgpass.conf)
python scripts/verify_state.py          # validates state/state.json against schema
```

To reset between rehearsals: `cp state/state.example.json state/state.json`.

## Demo 3 setup

```bash
cd demo3-fred-agent
cp .env.example .env   # fill in ANTHROPIC_API_KEY and FRED_API_KEY
pip install -r requirements.txt
python agent_reference.py    # full working version (fallback)
python agent_skeleton.py     # has two TODO blocks; live-coded during demo
```

Default model in `agent_skeleton.py` is `claude-sonnet-4-6`, overridable via `ANTHROPIC_MODEL`.

## Conventions worth preserving

- **Skill design pattern (used in demos 1, 2, 4, 5, 6):** every skill is a `SKILL.md` with YAML frontmatter (`name`, `description`, optional `args`), then phased sections: Pre / Act / Verify / Post. The `Verify` step is non-negotiable — it's the pedagogical point of the whole afternoon. The verify gate scales across demos: text grep (2) → computational DAG check via NetworkX (5) → JSON-schema + numerical-tolerance subagent (6).
- **Subagent persona files** live under `.claude/agents/<name>.md` with frontmatter `name`, `description`, `tools: [...]`. Personas are terse and direct ("You are X. Output exactly Y. Do not be polite.") because the orchestrator parses replies mechanically.
- **No commits from demo state.** The demos write into `tables/`, `summary/`, `data/`, `review/`, `state/state.json`. These are work products, not source. Don't commit them unless you are intentionally checkpointing demo output.

---
> Source: [velikov-mihail/edhec-master-class](https://github.com/velikov-mihail/edhec-master-class) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
