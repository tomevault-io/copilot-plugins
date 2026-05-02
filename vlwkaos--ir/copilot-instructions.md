## ir

> Package name on crates.io is `ir-search` (name `ir` was taken).

# ir — Agent Instructions

## Crate

Package name on crates.io is `ir-search` (name `ir` was taken).
Binary name is `ir`. See @Cargo.toml.

## Commands

```bash
cargo build                        # dev build
cargo build --release --bin ir     # release build
cargo test                         # unit tests (fast, no models needed)
cargo test -- --ignored            # includes LLM tests (require model files)
```

Benchmark runner (drives the real `ir` binary; requires BEIR dataset):
```bash
scripts/bench.sh fiqa              # bench current HEAD on FiQA
scripts/bench.sh fiqa v0.9.0       # compare HEAD vs v0.9.0
scripts/bench.sh miracl-ko         # Korean MIRACL benchmark
```
Results cached at `logs/results/{dataset}/{git7}.json` (gitignored).

## Environment Variables

| Var | Default | Description |
|-----|---------|-------------|
| `IR_EMBEDDING_MODEL` | auto-detect | Path to embedding GGUF |
| `IR_EXPANDER_MODEL` | auto-detect | Path to expander GGUF (qmd-1.7B) |
| `IR_RERANKER_MODEL` | auto-detect | Path to reranker GGUF (Qwen3-0.6B) |
| `IR_COMBINED_MODEL` | unset | Unified Qwen3.5 GGUF — replaces both expander + reranker (`IR_QWEN_MODEL` deprecated alias) |
| `IR_GPU_LAYERS` | `99` on macOS | Number of layers offloaded to GPU |
| `IR_FORCE_CPU_BACKEND` | unset | Set to `1` to disable Metal |
| `IR_LLAMA_LOGS` | unset | Set to `1` to enable llama.cpp verbose logging |
| `IR_MODEL_DIRS` | `~/local-models/` | Colon-separated extra model search dirs |
| `IR_CONFIG_DIR` | `~/.config/ir` | Override config/data dir. Supports `~` and `$VAR` expansion. |
| `XDG_CONFIG_HOME` | `~/.config` | **Deprecated** — use `IR_CONFIG_DIR` instead. Still works but emits a warning. |
| `IR_BENCH_SIGNALS` | unset | Research: emit `SIGNAL_FUSED\ttop\tgap` to pipeline log for threshold tuning |
| `IR_DISABLE_SHORTCUTS` | unset | Research: disable BM25 + fused strong-signal shortcuts for A/B benchmarking |
| `IR_FORCE_TIER1_ONLY` | unset | Research: force hybrid to return tier-1 fused results only (skip tier-2) |
| `IR_STRONG_SIGNAL_FLOOR_OVERRIDE` | unset | Research: override fused strong-signal floor threshold |
| `IR_STRONG_SIGNAL_PRODUCT_OVERRIDE` | unset | Research: override fused strong-signal product threshold |
| `IR_STRONG_SIGNAL_PRODUCT_PREPROCESSED_OVERRIDE` | unset | Research: override fused strong-signal product for preprocessed (Korean) collections |
| `IR_BM25_STRONG_FLOOR_OVERRIDE` | unset | Research: override BM25 strong-signal floor threshold |
| `IR_BM25_STRONG_GAP_OVERRIDE` | unset | Research: override BM25 strong-signal gap threshold |
| `IR_ALLOW_EXPANSION_WITHOUT_SCORER` | unset | Research: allow expansion without reranker (harmful in production: -0.53% nDCG on NFCorpus) |

Config dir precedence: `IR_CONFIG_DIR` → `XDG_CONFIG_HOME/ir` (deprecated) → `~/.config/ir`

Model search order: `IR_*_MODEL` env → `IR_MODEL_DIRS` → `~/local-models/` → `~/.cache/ir/models/` → `~/.cache/qmd/models/`

`QMD_EMBEDDING_MODEL`, `QMD_EXPANDER_MODEL`, `QMD_RERANKER_MODEL` are also checked as fallbacks.

All path env vars (`IR_CONFIG_DIR`, `IR_MODEL_DIRS`, `IR_*_MODEL`) support `~` and `$VAR`/`${VAR}` expansion.

Note: `IR_DIR` is set internally at startup (= resolved `ir_dir()` value). It appears in preprocessor commands stored in `config.yml` as `$IR_DIR/preprocessors/...` so they are portable. Not user-facing.

## Data Paths

- Config: `~/.config/ir/config.yml`
- Collection DBs: `~/.config/ir/collections/{name}.sqlite`
- Expander cache: `~/.config/ir/expander_cache.sqlite`
- Daemon socket: `~/.config/ir/daemon.sock`

## Architecture

### Search Pipeline

Three-tier escalation. Each tier runs only if the previous tier's result isn't strong enough.

| Tier | Models | Enables |
|------|--------|---------|
| 0 | none | BM25 (FTS5), in-process |
| 1 | Embedder | Vector, hybrid score-fusion (0.80·vec + 0.20·bm25) |
| 2 | Expander + Scorer | Query expansion (lex/vec/hyde → RRF) + reranking |

Strong-signal shortcut: raw BM25 top ≥ 0.75 AND gap ≥ 0.10 → skip Tier 1+2 entirely (`src/search/hybrid.rs:is_bm25_strong_signal`). Expander without scorer is a no-op (`hybrid.rs:112`).

See @research/pipeline.md for diagrams.

### Daemon Startup

Staged async: BM25 runs in-process immediately. Daemon starts in background.

- Tier 1 ready: embedder loaded → socket bound → client unblocks (waits up to 3s)
- Tier 2 ready: expander + reranker loaded → tier2 signal file written → client re-queries if needed (waits up to 7s)

Idle timeout: 3600s (configurable via `ir daemon start --timeout`).

## Known Gotchas

- **LLM tests are `#[ignore]`**: `cargo test` skips them. Run `cargo test -- --ignored` only when model files are present.
- **sqlite-vec must be registered before any connection opens**: `ensure_sqlite_vec()` uses `sqlite3_auto_extension` (process-global). Called once via `OnceLock` in `db/mod.rs`.
- **`LlamaBackend` is a singleton**: `OnceLock<LlamaBackend>` in `src/llm/mod.rs`. Loading a second model in the same process does NOT call `init()` again — this is intentional.
- **Daemon requires restart after binary change**: `ir search` auto-starts the daemon but won't restart a running one. Kill it with `ir daemon stop` after rebuilding.
- **`ir embed` prints "GPU context unavailable, falling back to CPU"** in sandboxed environments — normal, not an error.
- **Never run embedding or LLM inference in background shell tasks** — sandboxed shells have no Metal access, so they fall back to CPU and peg it. Hand these off to the user's terminal instead.
- **Strong-signal shortcut**: raw BM25 top ≥ 0.75 AND gap ≥ 0.10 skips all LLM work (`is_bm25_strong_signal`); fused top*gap ≥ 0.06 AND top ≥ 0.40 skips tier-2 (`is_strong_signal`). Both in `src/search/hybrid.rs`.

## Release

release.flow: rust-ci

CI (GitHub Actions) handles build, GitHub release, and Homebrew tap on tag push.

```bash
# Bump version
sed -i '' 's/^version = ".*"/version = "'"$VERSION"'"/' Cargo.toml
cargo check --quiet  # updates Cargo.lock

# Commit + tag + push (triggers CI release workflow)
git add Cargo.toml Cargo.lock CHANGELOG.md
git commit -m "v$VERSION"
git tag -a "v$VERSION" -m "v$VERSION"
git push origin main --tags

# Publish to crates.io
cargo publish   # publishes as ir-search
```

Prerequisites: `TAP_TOKEN` secret set in GitHub repo settings (used by CI to update Homebrew tap).

## good-to-go

- README.md + README.ko.md must both be updated for any user-facing feature (CLI flags, env vars, output formats)
- CHANGELOG.md Unreleased section must cover: new CLI flags, env var renames/deprecations, breaking behavior changes
- Enum variants in types.rs must be wired to a CLI flag or MCP field — check with `rg 'Variant::' src/ | grep -v test`
- Preprocessor protocol tests must use `cat` only — `rev` uses full stdio buffering in pipe mode on macOS and deadlocks. `tr`, `sed`, `sort` also buffer.
- IR_COMBINED_MODEL is the canonical combined-model env var; IR_QWEN_MODEL is a deprecated alias — do not promote the alias in new docs
- src/search/filter.rs must have unit tests for eval_clause + match_op — these are pure functions with no DB dependency; easy to test, and zero coverage is a gap [resolved v0.10.0: 9 tests added]
- FilterOp::Ne on multi-valued fields uses any-match semantics (same as all ops): `meta.tags!=rust` passes if ANY tag != "rust"; document this in README filter table, not just code comments [resolved v0.10.0: documented in both READMEs and tested]
- items_after_test_module: in Rust files, keep non-test items (impl fns, helper fns) BEFORE any #[cfg(test)] mod block — clippy::items_after_test_module will fail the build
- build_query_natural in db/fts.rs is used for all production BM25 queries; uses OR + stop word stripping for natural-language queries, AND for short keyword queries
- cargo clippy --all-targets -- -D warnings must pass before release; check llm/ files for needless_borrow when updating llama.cpp bindings
- warn_stale_preprocessor() in src/main.rs is a migration shim for ≤0.9.x users — removed at v0.13.0
- Research-only env vars (IR_BENCH_SIGNALS, IR_DISABLE_SHORTCUTS, IR_FORCE_TIER1_ONLY, IR_STRONG_SIGNAL_*_OVERRIDE, IR_BM25_STRONG_*_OVERRIDE, IR_ALLOW_EXPANSION_WITHOUT_SCORER) must NOT appear in README; CHANGELOG may name them only under "Dev / Benchmark Tooling"; document in CLAUDE.md env table
- preprocess.rs sentinel protocol (IRSENTINEL): process_line() sends content line + IRSENTINEL, reads until IRSENTINEL — prevents pipe deadlock when lindera emits no stdout for all-filtered lines (e.g. punctuation-only). Custom preprocessors must pass ASCII-only single-word lines through unchanged. When any preprocessor command changes (new binary, flags, or external tool replacing custom code): run probe `printf '.\n안녕하세요\ntest\n' | <new_command> 2>/dev/null | wc -l` — must equal 3, or WARN and confirm sentinel covers the 0-output case. Test suite must include at least one test where process_line() is called with a line the subprocess drops.
- IR_DIR is set internally at startup (= resolved ir_dir() value); appears in preprocessor commands as $IR_DIR/preprocessors/... for portability — do not expose in user-facing docs
- All path env vars (IR_CONFIG_DIR, IR_MODEL_DIRS, IR_*_MODEL) support ~ and $VAR expansion via expand_path() in src/config/mod.rs — tests for this must use ENV_LOCK mutex to prevent parallel env var interference
- scripts/preship.sh must pass (exit 0 or 2) before any signal-sweep run or release; run `--bm25-only` for fast CI gate, full for pre-release
- Default pool size for MIRACL-Ko signal sweeps: 50000 docs. Minimum stable floor from the variance study: 10000 docs. Do not use pool sizes <= 503 for between-seed variance decisions; those pools collapse to the mandatory qrel-linked docs and are deterministic.

---
> Source: [vlwkaos/ir](https://github.com/vlwkaos/ir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
