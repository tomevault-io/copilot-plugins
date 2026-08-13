## conproxy

> Cache proxy for heterogeneous RAG/vector search backends (Elasticsearch, OpenSearch, Qdrant, pgvector, Meilisearch, Pinecone, Milvus). Rust 2021, Axum + tonic, multi-process: lib + `conproxy` CLI + `test_runner` + `generate_embeddings` + `perf_summarize` + `hitrate_bench` + `console_snap` + `corpus_seed` + Python SDK.

# conproxy Agent Instructions

Cache proxy for heterogeneous RAG/vector search backends (Elasticsearch, OpenSearch, Qdrant, pgvector, Meilisearch, Pinecone, Milvus). Rust 2021, Axum + tonic, multi-process: lib + `conproxy` CLI + `test_runner` + `generate_embeddings` + `perf_summarize` + `hitrate_bench` + `console_snap` + `corpus_seed` + Python SDK.

## Product framing (for PR copy, README, and any external docs you write)

- **What:** retrieval-leg cache for **agentic RAG** (embed + upstream search). **Not** an LLM-answer cache.
- **Pitch:** cost + faster search on **hits**; agents re-query (retries, fanout, tool storms).
- **Not:** GPTCache / RedisVL SemanticCache territory; not "faster chat RAG" as the headline alone.
- **Proof:** `docs/benchmarks.md` + `make bench-hitrate*`; BYO with `make bench-hitrate-replay QUERIES=…`.
- **User-facing decision docs:** `README.md` (consider/skip + vs table), `docs/benchmarks.md`. This file is ops-focused.
- **Stability:** pinecone / milvus experimental; peer = trusted network, no mTLS.

## Fast Feedback Tiers

Three tiers — Tier 1 & 2 for the per-PR loop, Tier 3 for release publishing. Run the tier that matches your stage. Vertical-specific commands live in the `contributing` skill (Feature Test Matrix + 14 verticals).

## Local iteration loop

Default save-and-run:

```bash
make t                # cargo-nextest dev profile, falls back to cargo test --lib
make test-slow        # top-25 slowest tests
make test-filter PAT=foo
make target-prune     # drop conproxy-only build artifacts
```

`make t` warms to ~3-4 s on a 16-core host thanks to `cargo-nextest`, `lld` (`.cargo/config.toml`), and the dev-profile settings in `Cargo.toml` (`debug = "line-tables-only"`, `split-debuginfo = "unpacked"`). One-time: `make nextest-install`. Full rationale + measurements in `docs/dev-loop.md`.

### Tier 1: Smoke (every save, <60s)

```bash
cargo fmt -- --check
cargo clippy -- -D warnings
cargo test --lib
cargo test --features "embed-api" --lib
# optional lead proofs / gates:
# make proof-cascade
# make cov-scope-tune
# make sdk-smoke   # needs maturin + python3-dev
```

### Tier 2: Pre-PR Gate (before opening PR, ~8 min cold)

```bash
# 1. Format
cargo fmt -- --check

# 2-7. Lint (all feature surfaces)
cargo clippy -- -D warnings
cargo clippy --features "embed-api" --lib -- -D warnings
cargo clippy --features persistence --lib -- -D warnings
cargo clippy --features mcp --lib -- -D warnings
cargo clippy --features pgvector --lib -- -D warnings
cargo clippy --features linux-sandbox --lib -- -D warnings
cargo clippy --bin perf_summarize -- -D warnings
cargo clippy --features tokio-console --bin conproxy -- -D warnings

# 8-9. Test (lib unit tests)
cargo test --lib
cargo test --features "embed-api" --lib

# 10. Build (workspace — catches SDK proto breakage)
cargo build --workspace

# 11. Build (embed linker — ort deps, hard gate)
cargo build --features embed --lib

# 12-13. Check + test (e2e compile + pure-Rust e2e units)
cargo check --features e2e --tests
cargo test --features e2e --tests

# 14. Build (binary sanity)
cargo build --bin conproxy

# 15. MCP integration tests (conproxy mcp spawn + JSON-RPC)
cargo test --features mcp --test mcp_test
# optional Docker (label run-integration / nightly):
# make test-integration
# make e2e-smoke-core   # or e2e-cascade / e2e-federated
```

**Prerequisites:** `python3-dev` and Python headers required for `cargo build --workspace` (builds `sdk/python` cdylib). ONNX runtime libs auto-downloaded via `download-binaries` feature (`cargo build --features embed --lib`).

### Tier 2.5: PR CI gate (every PR + default branch)

`ci.yml` runs **six jobs** on every PR. `unit`, `integration`,
`integration-experimental`, `security`, and `fuzz` start in parallel;
`e2e` is gated on `unit` + `integration` so the heavy compose +
load + ignored suite never burns minutes if the cheaper gates would
block the merge.

| Job | Needs | Steps (summary) |
|-----|-------|-----------------|
| `unit` | — | fmt, clippy (all feature surfaces + bins), lib tests (default / embed-api / mcp / release), mcp_test, build (workspace + embed), release binary smoke, install-sim |
| `integration` | — | `make test-integration` (testcontainers: qdrant, ES, OS, meili, pgvector, cascade, peer, circuit, batch, metrics, context_config, singleflight) |
| `integration-experimental` | — | `make test-integration-experimental` (pinecone + milvus mocks, no real backends, ~2 min) |
| `security` | — | `cargo audit` (RustSec), `cargo deny check` (supply chain + license), `cargo clippy` with security lints (unwrap/expect/panic/indexing_slicing/arithmetic_side_effects) |
| `fuzz` | — | `cargo fuzz run` on all 5 targets, 60s each, `continue-on-error: true`. Crashes uploaded as `fuzz-artifacts` artifact. |
| `e2e` | unit, integration | Build release bin, `docker compose pull` (GHA cache), compose up, `test_runner wait all`, `load-data`, full ignored `e2e_proxy_suite` (all phases: smoke/health/query, cascade, load, reload, observability, efficiency, advanced, security), UAT (`e2e_uat`), security-focused e2e filter, compose down. Results uploaded as `e2e-results` artifact. |

`release.yml` (`v*` tag / `workflow_dispatch`) **publishes artifacts
only** — no test execution. By the time a tag lands, every commit on
the default branch has already passed the full `ci.yml` gate, so
`release.yml` skips straight to: cross-compile (x86_64-musl +
aarch64-gnu), aarch64 qemu smoke, multi-arch container (amd64 + arm64)
to GHCR, multi-arch manifest + `:latest` tag, OCI Helm chart to GHCR,
GitHub Release with cross binaries + chart `.tgz` attached.

### Tier 3: Release Gate (tag / manual only)

The full heavy gate runs in `.github/workflows/release.yml` on `v*` tags
and `workflow_dispatch`. Re-runs the PR release gate (guarantees a clean
green tree), then cross-compiles for `x86_64-unknown-linux-musl` and
`aarch64-unknown-linux-gnu`, runs a `--version` smoke under qemu on
aarch64, builds + publishes a multi-arch container image (linux/amd64 +
linux/arm64) to GHCR, builds + publishes the Helm chart to OCI on GHCR,
and attaches binaries + chart to a GitHub Release. Local mirrors of
these steps (PGO, DHAT, benches, ignored e2e, full integration) live in
Tier 3 below for reference. Expect ~25–35 min on a dev host.

Published artifacts per `v*` tag:

| Artifact | Where | Tags / version |
|----------|--------|----------------|
| Container | `ghcr.io/jmcgrath207/conproxy` | `<version>`, `v<version>`, `latest` (multi-arch) |
| Helm chart | `oci://ghcr.io/jmcgrath207/charts/conproxy` + `.tgz` on GH Release | `version` + `appVersion` stamped from tag |
| Binaries | GH Release + workflow artifacts | x86_64-musl, aarch64-gnu |

### Coverage (weekly, not a PR gate)

`.github/workflows/coverage.yml` runs `cargo tarpaulin` (lib + bins,
80% line gate via `make test-coverage-check`) on a weekly cron
(Monday 06:00 UTC), `push` to the default branch, and `workflow_dispatch`.
**Not** part of the PR gate — tarpaulin takes ~10 min cold cache and
runs best on a warm weekly schedule. Report is uploaded as the
`coverage-results` artifact.

Prerequisites: Docker daemon running; `cross` cargo subcommand (`cargo install cross`); targets `rustup target add x86_64-unknown-linux-musl aarch64-unknown-linux-gnu` (cross auto-installs).

```bash
# 1-4. Lint + test the release bundle
cargo clippy --features release --lib -- -D warnings
cargo test   --features release --lib

# 5. Build + smoke the shipped binary
cargo build --release --features release --bin conproxy
./target/release/conproxy --version
./target/release/conproxy --help 2>&1 | head -40   # top-level commands (no models)

# 6. Workspace release build (catches sdk/python breakage)
cargo build --workspace --release --features release

# 7. Release-bundle e2e + mcp_test
cargo test --features "release,e2e" --tests
cargo test --features release --test mcp_test

# 8. Default-profile cross-check (no drift)
make test

# 9. Known-gaps inventory refresh
make audit-known-gaps

# 10. Backend integration (prefer testcontainers) + multi-service e2e
# Prefer: make test-integration  (testcontainers; no bare docker run)
make test-integration
# Legacy bare docker run path (still OK for multi-service e2e load-data):
# docker run -d --name cqdrant -p 6333:6333 qdrant/qdrant
# docker run -d --name ces -p 9200:9200 -e discovery.type=single-node -e xpack.security.enabled=false docker.elastic.co/elasticsearch/elasticsearch:8.13.0
# docker run -d --name cpg -p 5432:5432 -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=conproxy_test ankane/pgvector:v0.5.1
# cargo run --bin test_runner --features default -- wait all
# cargo run --bin test_runner --features default -- load-data
cargo test --features "release,e2e" --test e2e_proxy -- --ignored
cargo test --features "release,e2e" --test e2e_uat   -- --ignored
# docker rm -f cqdrant ces cpg

# 11. Cross-compile + runtime smoke (requires binfmt_misc qemu registration)
docker run --privileged --rm tonistiigi/binfmt --install all   # one-time per host
cross build --release --features release --target x86_64-unknown-linux-musl --bin conproxy
cross build --release --features release --target aarch64-unknown-linux-gnu --bin conproxy
cross run   --release --features release --target aarch64-unknown-linux-gnu --bin conproxy -- --version

# 12. Install simulation (isolated --root)
cargo install --locked --path . --features release --root /tmp/conproxy-install
/tmp/conproxy-install/bin/conproxy --version
rm -rf /tmp/conproxy-install

# 13. Bench snapshot (bootstrap main baseline on first run)
make bench-save

# 14. DHAT heap profile smoke
make profile-dhat

# 15. PGO build (proper profile-guided optimization; ~10 min)
make profile-pgo

# 16. Optional perf diagnosis (CI-aware verdict, see .opencode/commands/perf-tuning.md)
make perf-tuning-quick

# 17. Optional hit-rate benchmark (cache-effectiveness, see docs/strategy-assessment.md §3)
make bench-hitrate        # fast exact-HR; bench-hitrate-sem / -onnx / -live go deeper
```

**Known gap:** `e2e_eval` requires Ollama OR `EVAL_PROVIDER=claude` API key — not run in this gate. Cross-compile runtime smoke (step 11) requires `docker run --privileged ... binfmt --install all` to register qemu-aarch64 binfmt on the host (re-run after host reboot). `Cross.toml` already handles protoc (musl) and glibc 2.30+ image (aarch64).

## Skills to Load

Trigger-based — load the first skill whose keywords appear in the task:

- **`contributing`** — keywords: `test`, `PR`, `CI`, `vertical`, `coverage`, `security-fuzz`, `onboarding`. Full test verticals (unit, e2e, eval, load, uat, bench, security, profiling, python sdk) + contributor workflow + PR checklist.
- **`rust-skills`** — keywords: `Rust`, `refactor`, `async`, `error`, `Result`, `unsafe`, `lifetimes`. Rust coding style.
- **`performance`** — keywords: `performance`, `optimize`, `slow`, `latency`, `alloc`, `flamegraph`, `bench`, `profile`, `pgo`, `dhat`, `hot path`, `tokio-console`, `metrics`, `cascade`. conproxy perf diagnostics (flamegraph, DHAT, tokio-console, metrics, bench, PGO) + `/perf-tuning` command.
- **`caveman-commit`** — keywords: `commit message`, `PR description`, `/commit`. Terse commit messages.
- **`k8s-dev`** — keywords: `tilt`, `kind`, `k8s`, `port-forward`, `address already in use`, `tilt up`, `helm conproxy`, `dashboard`, `e2e-k8s`, `HOST_IP`. Troubleshoot k8s dev loop (kind + Tilt + Helm + backends).

## Repo Rules

- `#![deny(unsafe_code)]` in `src/lib.rs:1` — every `unsafe` block needs `// SAFETY:` comment + `#[allow(unsafe_code)]`
- No new `unwrap`/`expect`/`panic` in non-test src (Cargo.toml `[lints.clippy]`)
- Locks never held across `.await`
- Public Result fns need `# Errors` doc section
- Cache writes are infallible (`CacheStore::insert` returns `()`); downstream must not assume insert was rejected. Internal failure modes (collision overwrite, eviction races) are silent by design.
- `Cargo.toml` features: `embed-api` (API providers), `embed` (ONNX), `mcp` (MCP server), `persistence` (redb), `pgvector`, `linux-sandbox`, `integration-tests` (testcontainers). Meta: `release` = `mcp` + `persistence` + `embed-api` + `pgvector` (ONNX + sandbox opt-in); `test` = `release` + `load-test` + `dhat-heap`.

### Don'ts (distilled from agent missteps)

- **Don't** add `unwrap`/`expect`/`panic` in non-test src — promote to `?` or `ok_or_else`
- **Don't** hold locks across `.await` — clone data out first, or use `parking_lot::Mutex` for non-async paths
- **Don't** fix pre-existing issues in unrelated PRs — keep diffs scoped to the change
- **Don't** pin test counts in assertions or docs — they drift; assert "all passed", not specific numbers
- **Don't** silence lints without an explanatory comment (e.g., `#[allow(clippy::unwrap_used)] // test module`)

## Known Gaps

Only the always-relevant essentials live here. Full technical list with workarounds is in the `contributing` skill.

- Default compose (`tests/e2e/docker-compose.yml`): qdrant + Elasticsearch + meilisearch×2 + postgres. Single-backend proof prefers testcontainers (`make test-integration`). Full matrix is nightly-friendly; PR optional (`run-integration` label). Experimental: `make test-integration-experimental` (pinecone/milvus).
- **ONNX `embed`:** `cargo build --features embed --lib` is the hard gate. Uses `download-binaries` feature — ort-sys auto-downloads prebuilt ORT libs on first build (~50MB, cached). No system ORT install required. Not a Tier-3 full-test blocker.
- **`e2e_eval`:** requires Ollama OR `EVAL_PROVIDER=claude` API key — **out of** Tier-3 release gate (non-blocking).
- Inherits global caveman mode (terse responses, drop articles/filler)
- Run `make audit-known-gaps` to surface the current pre-existing warning inventory (clippy unwraps, bench compile status, unallowed warnings)
- Eval llama.cpp path: `make eval-llamacpp` needs llama-server + GGUF (direct; not passthrough). Tilt + kind: see `k8s-dev` skill for full troubleshooting (port strategy, Tiltfile kwargs, docker build, HOST_IP, dashboard). `docs/k8s-dev.md` for human-readable dev loop.
- **`linux-sandbox`:** Linux-only; not in `release` meta-feature.
- **Agents Phase B** (done): file reload rebuilds `AgentRegistry` from `[[agents]]` in config (ArcSwap). File is authoritative on reload — API-only agents are dropped. `api_key` / `rate_limit` rotate without restart. API mutations survive until next file reload. Full IAM / OIDC not in scope.
- **Peer mesh v1:** LWW by wall timestamp. Optional `[proxy.peer] shared_secret` requires `x-peer-secret` on peer gRPC; default off = trusted network only. **No mTLS (not planned)** — use NetworkPolicy / mesh sidecar externally if needed. Process `api_key` still applies to non-peer gRPC when set.

---
> Source: [jmcgrath207/conproxy](https://github.com/jmcgrath207/conproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
