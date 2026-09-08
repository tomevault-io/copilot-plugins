## dejadb

> Embedded memory engine for AI agents — reference implementation of OMS (Open

# DejaDB

Embedded memory engine for AI agents — reference implementation of OMS (Open
Memory Spec). Rust workspace of 12 crates (plus `dejadb-js`, a standalone napi
package built outside the workspace). Memories are immutable
content-addressed grains in per-file Turso databases, queried with CAL, and
rendered into model-ready context in-process (no server in the recall path).

**Status**: published — the library crates + the `deja` binary on crates.io,
`dejadb` on PyPI and npm (`dejadb-py` and `dejadb-bench` stay `publish = false`).
The version lives in `[workspace.package]` in the root `Cargo.toml` (all crates
inherit it) and `CHANGELOG.md` records each release — don't restate the number
here (it goes stale). `ARCHITECTURE.md` is the design source of truth — the
architecture and the numbered design decisions; `CHANGELOG.md` summarizes what
exists; `crates/dejadb-bench/RESULTS.md` has the benchmark numbers.

## Commands

```bash
cargo test --workspace            # full suite (~950 tests, fast)
cargo test -p dejadb-cal          # per-crate
cargo run --release -p dejadb-store --example bench       # latency gates
cargo run --release -p dejadb-store --example voice_loop  # 50ms-cadence gate
cargo run -p dejadb -- recall --db demo.db --ns caller --subject john
```

- **Do not run blanket `cargo fmt`** — the tree is not uniformly rustfmt-clean
  (~177 files differ). Match surrounding style; format only
  the lines you touch.
- If CLI/MCP smoke tests fail with "spawn dejadb: No such file or directory":
  the cached test binary has a stale absolute path baked in via
  `CARGO_BIN_EXE_dejadb` (happens after the repo folder moves/renames).
  Fix: `touch crates/dejadb-cli/tests/*.rs` and re-run.
- CI (`.github/workflows/ci.yml`): test on ubuntu/macos/windows, clippy
  (`-D warnings`), MSRV build, `cargo doc`, coverage, Python (maturin + pytest),
  and Node (napi build + `node --test`). `security.yml` runs `cargo deny`.
  Still run tests locally before pushing.

## Workspace (dependency order)

```
memory stack:  dejadb-core ← dejadb-store ← dejadb-cal ← dejadb-context ┐
loop engine: deja-loop ← dejadb-loop (adapter) · dejadb-llm (providers) ┤
                              both feed the leaf crates ↓                 ↓
             dejadb-mcp, dejadb-server, dejadb-py, dejadb (binary), dejadb-bench
```

| Crate | What | CLAUDE.md |
|---|---|---|
| `dejadb-core` | `.mg` format, canonical serialization, content addressing, 12 grain types, tool-schema rendering | yes |
| `dejadb-store` | The store: dictionary-encoded triples, hybrid recall, heads/forks, bundles, CAS blobs (encrypted under an HKDF-derived subkey when the memory is), DSAR `subject_report`, declarative `retention:<ns>` policies, memory-tool adapter, migration importers. Backend-agnostic logic over an internal `Db` seam — embedded Turso (default) or PostgreSQL (`feature = "postgres"`, one memory = one schema, advisory-locked single writer, pgvector) | yes |
| `dejadb-conformance` | Backend-parameterized conformance suite (`publish = false`) — one case list (forks, replication, tombstones, PITR, BM25, vectors, CAS, CAL smoke) run against BOTH backends; the Pg runner needs `DATABASE_URL`/`DEJADB_PG_URL` and hard-fails when `CI=true` without one | — |
| `dejadb-cal` | CAL lexer/parser/executor, ASSEMBLE, `DejaDbFacade` + mounts | yes |
| `dejadb-context` | Budget-aware SML/TOON/Markdown/JSON rendering | yes |
| `deja-loop` | Substrate-agnostic self-improvement engine: `OmsSubstrate`/`LlmBackend` traits, 12 analyzers (incl. default-off `retention_sweep`), four gates, recommendation lifecycle, LLM DISCOVER→GROUND→VERIFY verifier, outcome measurement (no DejaDB deps) — `docs/loop.md` | — |
| `dejadb-loop` | DejaDB substrate adapter for Deja Loop (`deja_loop::OmsSubstrate` over `DejaDbFacade`) + recall-telemetry sidecar | — |
| `dejadb-llm` | Out-of-box LLM backends (OpenAI-compatible/Anthropic/Ollama over a small blocking HTTP client) for Deja Loop + the `remember()` free-text→Fact extraction (`extract.rs`) | — |
| `dejadb-mcp` | Stdio MCP server (see below) | — |
| `dejadb-server` | Web console + dejad hub (see below) | — |
| `dejadb` | The `deja` binary (see below) | — |
| `dejadb-py` | PyO3 bindings (see below) | — |
| `dejadb-bench` | Reproducible benchmark harnesses (latency, honesty, LoCoMo accuracy) | — |
| `dejadb-js` | Node (napi) bindings — **standalone package, not a workspace member** (see below) | — |

## Cross-cutting invariants

1. **Grains are immutable and content-addressed** (SHA-256 over the whole
   `.mg` blob). Nothing ever edits a stored blob; every edit is a
   supersession, every removal a tombstone (`forget`) or crypto-erasure.
   Store code mutates the *index layer* only.
2. **Canonical serialization is frozen** (NFC, sorted keys, compact keys,
   omit-defaults). Changing it silently changes every content address and
   breaks OMS conformance — see `crates/dejadb-core/CLAUDE.md`.
3. **CAL destruction is authorization-gated, not structural** (CAL 1.3,
   [`docs/cal-all-you-need-proposal.md`](docs/cal-all-you-need-proposal.md)).
   The destructive statements are `FORGET <hash>` (single-grain tombstone,
   `delete` verb), `FORGET SUBJECT "<id>" [WITH text_mentions]` (identity
   erasure, `erase` verb), and `PURGE OLDER THAN <n><d|h|m> [TYPE t]`
   (retention sweep, `erase` verb) — BECAUSE mandatory on the latter two,
   optional-but-recorded on the hash form; every execution writes a Tier-2
   audit Observation in `agent:authz`. Grants live in the file as
   `mg:permits` Facts (`dejadb_core::authz`); the session's `AuthzSet`
   decides, and `CalExecutorConfig::allow_destructive_ops` remains a
   process-wide restrictive **cap** over any grant (`--no-destructive-ops`).
   Destruction takes a hash, an identity, or an age — never a predicate:
   `DELETE`/`ERASE`/`TRUNCATE`/… stay lexer-blocked non-tokens, `FORGET
   USER/SCOPE` stay text-refused, `DROP` accepts only TEMPLATE/QUERY, and
   saved-query bodies stay read-only. Statement classification has ONE
   source of truth (`dejadb_cal::classify`, exhaustive, no wildcard).
   [`docs/erasure.md`](docs/erasure.md) records the erasure requirements;
   its former "out of CAL" deviation is retired by CAL 1.3. Audit records
   name a subject **fingerprint** (`authz::subject_fingerprint`), never the
   identity — an immutable, replicating grain naming the erased subject
   would undo the erasure it records. The read-only mirror is `REPORT
   SUBJECT` (classifies `Read`, `read`-gated, not behind the destructive
   cap): the report and the erasure share ONE selector, so a DSAR discloses
   exactly what an erasure removes. [`docs/gdpr.md`](docs/gdpr.md) is the
   article→capability map.
4. **CAL syntax is an OMS conformance contract** — no new CAL syntax
   without a spec-level decision.
5. **One memory = one isolation unit** — a file on the embedded backend, a
   Postgres schema on the `postgres` backend; either way it is the unit of
   erasure, sync, portability, and write parallelism. Single writer per
   memory — enforced on BOTH backends: an advisory lock on Postgres, and on
   the embedded backend a process-wide open-path registry so a second handle
   on one file fails at open (`STO-E002`) instead of silently drifting its
   cached allocators and corrupting the first handle's writes. Rust/Python
   release on drop; Node calls `close()`. Adding a grain that is already
   stored is a no-op returning the existing hash, not an error.
   Cross-memory queries go through
   ASSEMBLE with facade mounts, not shared connections. Files are
   self-describing: the `meta` table carries file-truths (`text_index`,
   `entity_relations`, embedding provenance) and CAL host metadata — saved
   queries and custom templates ride there as `qry:<name>`/`tpl:<name>` rows,
   so they travel with the file rather than living in any one client. Bare `open()` honors them;
   `open_with()` deliberately re-stamps and reports changes via
   `open_warnings()`. Host config (embedder capability, executor limits) is
   per-process and never persisted in the file.
6. **Dependency-light by policy**: no clap (hand-rolled args), no HTTP
   framework (std `TcpListener`), no MCP SDK (hand-rolled JSON-RPC), no
   workspace-wide async runtime (store wraps a private tokio current-thread
   runtime behind a sync API). Think twice before adding a dependency.

## Error codes

Every user-facing error carries a stable `DOMAIN-Ennn` code (3-letter
uppercase domain, `-E`, digits) as the **leading token of its `Display`
string**, plus a `code()` method. Domains: `FMT` (.mg format), `MEM`
(grains + tool-schema binding), `STO` (Turso store), `CRY` (crypto), `VAL`
(input validation), `CAL` (query language), `SYS` (internal). A reported code
alone locates the variant and subsystem. **Codes are append-only** — never
renumber or reuse one. Source of truth for text is inline on `DejaDbError`
(`dejadb-core/src/error.rs`), `SchemaSubsetError`, and `CalError`
(`dejadb-cal/src/errors.rs`); the full registry + the rule for adding one is
[`ERROR_CODES.md`](ERROR_CODES.md). Format/uniqueness are test-enforced
(`error_code_tests`, `test_all_error_codes_have_unique_codes`).

## Smaller crates

- **dejadb-mcp**: 14 tools (`dejadb_recall/add/supersede/forget/remember/cal`,
  the DSAR read `dejadb_subject_report`,
  the graph/time reads `dejadb_related/entity_at/step_actions`, the
  run<->memory join `dejadb_run_trace/runs_touching`, and the loop pair
  `dejadb_loop/recommendations`)
  over newline-delimited JSON-RPC 2.0 on stdio, protocol rev `2025-06-18`.
  Convention: tool failures are `isError: true` *results*; only protocol
  errors are JSON-RPC errors. Notifications (no id) get no response. No
  in-crate tests — exercised by `dejadb-cli/tests/mcp_smoke.rs`, which drives
  the real binary over real stdio.
- **dejadb-server**: hand-rolled std-only HTTP/1.1, one request per
  connection. `ui` console binds loopback and is **unauthenticated by
  default**; `with_auth(token)` (CLI `deja ui --token-env VAR`) requires the
  token on **every** request — browsers via the native HTTP Basic prompt (any
  username, password = token), scripts via `Authorization: Bearer` — and a 401
  carries `WWW-Authenticate: Basic` so browsers prompt. `into_hub(token, dir)`
  is the separate hub mode (CLI `deja hub`, where `--token-env` is
  **mandatory**): bearer auth on POSTs + the `/api/segment*` surface, which is
  gated on **reads too** — only the non-segment reads are open. Base64 for
  Basic is hand-rolled (no dep). Body cap 1 MiB. Cross-origin POSTs are
  rejected via Origin check (drive-by protection). The console is
  one embedded HTML file (`console.html`, vanilla JS, no build step) — a
  plain-language memory browser with an interactive graph, the loop review
  queue, and a Developer-mode toggle that reveals hashes/op-log/CAL; design
  source of truth is the Paper file "DejaDB", page "Console v2 — Redesign".
  Read-only `GET /api/config` reports effective config + file-vs-host
  reconciliation warnings.
  `tests/multichannel_tests.rs` is the §8 acceptance test (voice + WhatsApp +
  email sharing one memory via the hub).
- **dejadb**: ~29 verbs (incl. `hub`, `migrate` from other memory systems,
  `reindex`, the graph/time reads `related`/`entity-at`/`step-actions`, the
  join `run-trace`/`runs-touching`, and the DSAR read `subject-report`),
  hand-rolled `parse_args` → HashMap; global `--embed-cmd` installs
  a `CommandEmbed` for vector recall on any verb. Opens honor
  the file's meta declarations; `--index-text true|false` explicitly
  re-stamps; open warnings print to stderr.
  `hook claude-code` only *prints* the settings snippet (never writes user
  config); `capture-stop` reads Claude Code hook JSON from stdin and stores
  the last exchange as thread-indexed Events.
- **dejadb-py**: `#[pyclass] DejaDB` over `DejaDbFacade`. FFI convention:
  **scalars in, JSON strings out**; errors → `PyValueError`. abi3-py39
  cdylib; build with maturin (`build.rs` handles macOS
  `-undefined dynamic_lookup` for bare cargo builds).
- **dejadb-js**: `#[napi]` methods over `DejaDbFacade`. Same **scalars in, JSON
  strings out** convention as `dejadb-py`; native Node addon via napi-rs (not
  wasm). Standalone package — **not** a `cargo` workspace member, so
  `cargo test --workspace` skips it; CI's `node` job builds it with
  `napi build --release` and runs `node --test __test__/smoke.mjs`.

## Local artifacts (gitignored, don't commit or rely on)

`demo.db*` and `*.db/-wal/.blobs` (scratch memories), `m0-data/` (spike
outputs), `name-reservation/` (registry placeholder stubs), `target/`.

## Naming

Brand "DejaDB", CLI binary `deja` (package/crate `dejadb`), hub daemon
"dejad", Python module `dejadb`. The OMS spec itself is external (CC0); OMS
conformance is the
compatibility mechanism with other implementations.

---
> Source: [AreevAI/dejadb](https://github.com/AreevAI/dejadb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
