## lethe

> Self-improving memory store for LLM agents. BM25 + dense vector hybrid

# lethe — agent guide

Self-improving memory store for LLM agents. BM25 + dense vector hybrid
retrieval with cross-encoder reranking and clustered retrieval-induced
forgetting (RIF). Indexes agent transcripts (Claude Code / Codex) directly
— no capture hooks, no summarization. Ships as a single Rust binary (`lethe`)
plus PyPI / npm bindings.

## stack

- **Rust workspace** (`crates/`) — production. Toolchain pinned via
  `rust-toolchain.toml`.
  - `lethe-core` — library: tokenize, bm25, faiss_flat, rrf, dedup, rif,
    kmeans, encoders (ONNX via `ort`), DuckDB persistence, npz reader,
    memory_store, union_store, markdown_store, transcript_store.
  - `lethe-cli` — `lethe` binary (clap; embeds the TUI).
  - `lethe-tui` — ratatui library called from the CLI on no-arg invocation.
  - `lethe-py` — PyO3 binding → PyPI `lethe-memory`.
  - `lethe-node` — napi-rs binding → npm `lethe`.
  - `lethe-benchmark` — internal parity bench helper (`publish = false`).
- **DuckDB** for entry metadata + embedding BLOBs (single source of truth).
- **ONNX Runtime** (via `ort`) for the bi-encoder + cross-encoder.
  Default models: `Xenova/all-MiniLM-L6-v2`, `Xenova/ms-marco-MiniLM-L-6-v2`.
- **Python `research_playground/lethe_reference/`** kept for the research-journey trail: produces the
  benchmark numbers cited below. Installed as `lethe-memory-legacy` in
  dev venvs; not published.
- **Datasets** (`tmp_data/`, gitignored): LongMemEval (S), NFCorpus.

## commands

```bash
# Rust workspace
cargo build --workspace --release        # → target/release/lethe
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all -- --check

# Pre-commit / pre-push hooks (replaces the deleted ci.yml)
cargo install --git https://github.com/j178/prek
prek install                             # writes .git/hooks/pre-commit + pre-push
prek run --all-files                     # one-off run

# Python reference impl (research trail; library only)
uv pip install -e research_playground/lethe_reference/
cd research_playground/lethe_reference && uv run pytest tests/ -v     # 148 + 8 PyO3 parity = 156
uv run python research_playground/baseline/run.py

# Python ↔ Rust parity bench
uv run python research_playground/rust_migration/prepare.py
uv run python research_playground/rust_migration/longmemeval.py --compare
uv run python research_playground/rust_migration/components.py --compare
uv run python research_playground/rust_migration/latency.py --compare

# CLI surface (the `lethe` binary)
lethe                                    # no args → TUI (in a terminal)
lethe index                              # index this project's transcripts
lethe index --all                        # reindex every registered project
lethe search "query" --top-k 5           # reindexes changed transcripts first
lethe search "query" --all --top-k 5     # all registered projects (per-project read-only)
lethe projects list|add|remove|prune
lethe expand <chunk-id> [<chunk-id> ...]
lethe status
```

`tree -L 2 -I 'target|.venv|node_modules|tmp_*|results|.git|.lethe'` if
you want the current directory layout.

## key architecture decisions

- BM25 + FAISS-equivalent hybrid retrieval (BM25 is the strongest single
  signal on conversation data).
- Cross-encoder rerank on the merged candidate pool; adaptive depth
  (shallow `k=30`, deep `k=100` only when shallow confidence is low —
  see `research_playground/deep_pass/results/BENCHMARKS_DEEP_PASS.md`).
- Cosine 0.95 dedup on add (removes ~5% of LongMemEval, +6.5% NDCG).
- Tier lifecycle: naive → gc → memory (with decay and apoptosis).
- RIF: retrieval-induced forgetting suppresses chronic false positives
  at the candidate-selection stage (workload-specific to long-term
  conversational memory).
- DuckDB + npz interop for persistence; no external services.
- Embeddings are *the* source of truth in DuckDB. `MemoryStore::open()`
  rebuilds the in-memory `FlatIp` from BLOBs every cold start
  (~100 ms on a 200k corpus); no separate FAISS index file is written.

## benchmark headline

LongMemEval S (200k turns, 500 questions, 200-query eval sample):

| System | NDCG@10 |
|---|---|
| Vector only | 0.1376 |
| BM25 only | 0.3171 |
| Hybrid RRF | 0.2408 |
| **Hybrid + cross-encoder rerank** | **0.3817** |

Full methodology: `BENCHMARKS.md`. 18-checkpoint research journey:
`RESEARCH_JOURNEY.md`. arXiv preprint source: `paper.tex`.

## docs / READMEs

The root `README.md` is the project landing page (linked from each
per-crate README). Each publishable crate also has its own README
that gets surfaced on the registry it ships to:

| File | Surfaces on |
|---|---|
| `crates/lethe-core/README.md` | crates.io `lethe-core` |
| `crates/lethe-cli/README.md`  | crates.io `lethe-cli` |
| `crates/lethe-tui/README.md`  | crates.io `lethe-tui` |
| `crates/lethe-py/README.md`   | PyPI `lethe-memory` |
| `crates/lethe-node/README.md` | npm `lethe` |
| Root `README.md`              | GitHub repo landing page |

When you edit the root `README.md`, **always check whether the change
should propagate to one or more per-crate READMEs.** Examples that
need to fan out:

- Install instructions changed → update `lethe-cli/README.md`,
  `lethe-py/README.md`, `lethe-node/README.md`.
- Top-level usage example tweaked → update the matching per-binding
  example.
- New benchmark headline number → update the table referenced from
  each README's "see also" block (or just the root, if the per-crate
  READMEs only link to the root for numbers — current default).
- Architecture diagram or feature list change → update at minimum the
  `lethe-core/README.md` since library users see it first.

If a change is purely landing-page content (badges, plugin GIFs,
"who is this for"), it stays in the root only.

## git + release

- Conventional commits. release-please bumps version on `feat:` /
  `fix:` only — use either when you want the next release to fan out
  to crates.io / PyPI / npm / Homebrew. Everything else (`chore:`,
  `docs:`, `refactor:`, `test:`, `perf:`) does not trigger a release.
- **Never hand-edit changelogs or version numbers.** release-please owns
  both: the `CHANGELOG.md` / `plugins/CHANGELOG.md` files, the workspace
  version and every mirrored version (the `extra-files` in
  `release-please-config.json` — crate manifests, `package.json`,
  `pyproject.toml`, plugin manifests, marketplace manifests,
  `.release-please-manifest.json`). A manual bump is overwritten on the
  next run and only causes churn. To move the version, land the right
  conventional commit and let the release PR bump it:
  - `fix:` → patch, `feat:` → minor, `feat!:` / a `BREAKING CHANGE:`
    footer → breaking. **`bump-minor-pre-major: true` is set**, so while
    the project is pre-1.0 a breaking change bumps *minor*, not major
    (0.14.0 → 0.15.0); it does not jump to 1.0.0 on its own. Remove that
    flag (or land a manual `release-as`) when you deliberately want 1.0.
  - The **squash-merge commit title** is what release-please parses, so
    the PR title must carry the right type/`!` — individual branch
    commits are discarded on squash.
- Workspace version is pinned at `Cargo.toml :: [workspace.package].version`;
  every member crate inherits via `version.workspace = true`. Same value
  is bumped in `crates/lethe-py/pyproject.toml` and
  `crates/lethe-node/package.json` via release-please's `extra-files` /
  linked-versions plugin.
- Multi-platform release artifacts are built **locally** via
  `scripts/release/build.sh` (host / `--macos` / `--all`) and uploaded
  to the GitHub Release as assets. The `release-rust.yml`,
  `release-pypi.yml`, `release-npm.yml` workflows then publish from
  those assets — they do **not** run their own build matrices.
- **Intel Mac (`x86_64-apple-darwin`) is not a supported target.**
  Upstream `ort` dropped Intel Mac support in rc.11 (changelog at
  https://github.com/pykeio/ort/releases/tag/v2.0.0-rc.11), and
  there is no version going forward that ships a prebuilt ONNX
  Runtime for it. Apple Silicon and Intel/ARM Linux/Windows are
  supported.
- Required repo secrets for a successful publish:
  - `CARGO_REGISTRY_TOKEN` — crates.io API token, scoped to `lethe-*`
  - `NPM_TOKEN` — npm Granular Access Token, scoped to `lethe`
  - `HOMEBREW_TAP_GITHUB_TOKEN` — fine-grained PAT, Contents R/W on
    `teimurjan/homebrew-lethe`
  - PyPI uses Trusted Publishing (no token; configure on PyPI side
    plus a GitHub `pypi` environment).

---
> Source: [teimurjan/lethe](https://github.com/teimurjan/lethe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
