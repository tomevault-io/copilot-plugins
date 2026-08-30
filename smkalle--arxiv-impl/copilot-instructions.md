## arxiv-impl

> Multi-paper workspace: each subdirectory is an independent implementation of one arXiv paper. There is no root-level application, package, or test suite. **`cd` into the relevant subproject and follow its local `AGENTS.md` / `CLAUDE.md`.** `README.md` holds the canonical table of contents (a subset of subprojects are listed there; others are listed below).

# AGENTS.md — arxiv_impl (monorepo root)

Multi-paper workspace: each subdirectory is an independent implementation of one arXiv paper. There is no root-level application, package, or test suite. **`cd` into the relevant subproject and follow its local `AGENTS.md` / `CLAUDE.md`.** `README.md` holds the canonical table of contents (a subset of subprojects are listed there; others are listed below).

## OpenWiki

This repository has documentation located in the /openwiki directory.

Start here:
- [OpenWiki quickstart](openwiki/quickstart.md)

OpenWiki includes repository overview, architecture notes, workflows, domain concepts, operations, integrations, testing guidance, and source maps.

When working in this repository, read the OpenWiki quickstart first, then follow its links to the relevant architecture, workflow, domain, operation, and testing notes.

## Subprojects

| Folder | Paper | Local docs | Status |
|---|---|---|---|
| `sira/` | SIRA (2605.06647) — ticket→KB retrieval via LLM enrichment + weighted BM25 | `AGENTS.md`, `CLAUDE.md`, `sira-ticket-kb-spec.md`, `ITERATIONS.md` | Iter 1–5 coded; eval set is placeholder data |
| `AAFLOW-2605.02162/` | AAFLOW (2605.02162) — zero-copy Arrow-native ticket ingest → FAISS | `AGENTS.md`, `CLAUDE.md`, `*-spec-*.md`, `Makefile`, `pyproject.toml` | v0.1.0 release-ready |
| `datamaster/` | DataMaster (2605.10906) — e-commerce catalog enrichment agent | `CLAUDE.md`, `CATALOGAGENT_SPEC.md` | v0.1.0; 42 tests pass |
| `jina-2605.08384/` | Jina (2605.08384) — enterprise knowledge search prototype (`ek_search/`, FastAPI) | `AGENTS.md`, `TECH_SPEC.md`, `ITERATION_PLAN.md` | Prototype |
| `sira-kb-ingestor/` | SIRA KB ingestor package | `README.md`, `RELEASE_v0.1.0.md`, `pyproject.toml` | v0.1.0 |
| `tkmem-2605.13941/` | TKMEM/EvolveMem (2605.13941) — TicketMind retrieval prototype | `AGENTS.md`, `ITERATIONS.md`, `ticketmind-spec.md` | Spec + partial; impl lives under its nested `sira/` |
| `evolvemem-2605.13941/` | EvolveMem (2605.13941) — EvolveMem TicketMind v2.0 runner: L1→L2→L3→L4 AutoResearch loops over the SIRA pipeline | `pytest.ini`, `requirements.txt`, `evolvemem_runner.py`, `api.py` | Implementation present; tests defined |
| `rgqm/` | Red Queen Gödel Machine (2606.26294) — EpochForge Lite prototype | `AGENTS.md`, `rqgm-gemini-spec.html` (PRD), `rqgm-visual-explainer.html` | Implementation present; erasure-invariant test passing |
| `orchestrator/` | **metaorch** — meta-orchestrator pipeline chaining all 8 subprojects' use cases end-to-end via minimal contract-bound adapters (FastAPI + Pydantic v2; in-memory fakes, no sibling imports) | `AGENTS.md`, `SPEC.md`, `README.md`, `pyproject.toml`, `scripts/live_test.py`, `streamlit_app.py` | v0.1.0; 120 tests pass; optional Streamlit admin console |

Each subproject is self-contained: its own deps, own test command, own run location. Several subprojects consume each other (e.g. `AAFLOW` and `sira-kb-ingestor` feed `sira/`); confirm cross-project contracts in the consuming project's spec before changing a producer's output schema.

## Root conventions

- **Use `python3`, not `python`** (`python` is not on PATH anywhere in this repo).
- **No root CI, no root `pyproject.toml`, no root `requirements.txt`, no root test/lint command.** Each subproject defines its own. Do not add a root-wide test runner.
- **Run commands from inside the subproject folder**, never from the repo root. Several projects insert parent paths into `sys.path` or mount templates relative to CWD.
- **Generated artifacts are gitignored** — BM25 pickles, `df_store.json`, enriched corpora, FAISS indexes, Arrow files, `archive.json`, `.env`, venvs, `.pytest_cache`. Do not commit these. Per-project ignore rules live in the root `.gitignore` (e.g. `sira/data/*.pkl`, `datamaster/catalog-agent/artifacts*/`). Extend that file when adding a new generated-artifact path.
- **Commit style** (from `git log`): concise imperative subjects, one paper concern per commit (e.g. `Add sira kb ingestor package`, `Stop tracking generated SIRA artifacts`). No conventional-commits prefix.
- **No git submodules.** Each subproject is a plain top-level directory in one repo.

## Where to look first in a subproject

Priority order for ramp-up: the subproject's `AGENTS.md` (or `CLAUDE.md`) → its spec/`*-spec*.md` → its `README.md`/`ITERATIONS.md` → executable config (`pyproject.toml` / `Makefile` / `pytest.ini`). If docs conflict with code or config, trust the executable source.

---
> Source: [smkalle/arxiv_impl](https://github.com/smkalle/arxiv_impl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
