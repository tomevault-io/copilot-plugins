## imca

> This is the standalone code for the Inspectable Memory paper. Keep it independent

# Research artifact guide

This is the standalone code for the Inspectable Memory paper. Keep it independent
of plugin implementations, external memory services, provider configuration and
credentials. Never copy a broader development repository or its Git history here.

- Runtime: Python 3.11–3.13. Demo and LoCoMo use stdlib/SQLite; code ranking uses NumPy.
- Validate with `uv run --frozen python -m pytest -q` and the documented reproductions.
- Add tests for changes, including real SQLite multi-save and HTTP behavior.
- Keep `data/` and historical `results/` immutable; new output goes in `outputs/`.
- Public FastAPI source in `data/code/source` has its own MIT license. Do not edit it.
- Retrieve without labels; freeze top-5 before scoring. An individual turn consumes
  an individual result slot. Never infer a subject from a context link.
- Calendar rescue requires a unique attributed event, contained interval and matching modality.
- All LoCoMo conversations were used for development; no held-out or no-loss guarantee.
- Keep recorded dense rank input distinct from fresh BM25 fallback. No new model calls
  are made by the release; historical reader output applies only to its original top-5.
- Temporal fixtures are authored extensions. Valid time is a pinned linear snapshot
  position; recorded time is UTC. Preserve evidence, scopes and historical assertions.
- `/` and `/live` are real-write workspace labs; `/replay` preserves the frozen
  artifact. `live_memory.py` wraps, but does not change, frozen memory rules.
  Workspaces isolate data by scope, assertions are append-only, and only explicit
  replacement closes validity. Keep actual UTC recording time distinct from snapshot
  position. Guided agents are source labels, not LLM calls. Preserve loopback/Origin
  protection and never present workspace identifiers as public authentication.
- Review staged files for local paths, archives, credentials, unrelated code and source
  provenance before publication. Preserve all upstream licenses.
- Additional benchmark adapters are memory-isolated development diagnostics. Keep raw
  MemoryAgentBench/HaluMem data in ignored `outputs/`, freeze top-5 before labels, and
  never present the HaluMem oracle-writer result as leaderboard-comparable QA.

---
> Source: [nicl-nno/IMCA](https://github.com/nicl-nno/IMCA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
