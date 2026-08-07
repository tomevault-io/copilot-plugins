## rbitcoin

> Composition (has-a) is typically more clear and less error prone than

# Agent notes

## Prefer composition over inheritance in data models

Composition (has-a) is typically more clear and less error prone than
inheritance (is-a), although rust traits can make that blurry. Avoid tall
inheritance trees.

## Prefer immutable data structures

Immutable data structures built once then composed with other immutable
structures typically perform better than data structures mutated over time.

For example, prefer an immutable map that is built from the streamed results
of some work over a mutable hashmap. Even if the data structure itself is
mutable, in rust not having to make it mutable makes life better.

If we needed to add additional data to the members of a hashmap, we could
create an outer map that contains the additional info and annotates the
members of the inner map on read, converting them to the data type with the
additional fields (which will also have the inner object).

In short, prefer composition over mutation as well as composition over
inheritance.

## Store concurrency: lock-free by default

**Default: no locks on the store hot path.** Concurrency is **roles + publish
order + HWM**, not map mutexes (maps removed — phase 6).

| Rule | Detail |
|------|--------|
| Roles | At most **one Class A appender** and **one spend annotator** per process; **N readers** of published ranges always free |
| Publish | body → idx → count/HWM (Release); then head / `header_txs` as visibility requires |
| Capacity grow | fallocate/`set_len` only (no map epochs); readers use published HWM |
| Layout grow (`tx.head`) | **segment roll**: seal open head (fuse8) + create new fixed 25-bit head — no mono-file bits-widen |
| Class C tip | L2 write-behind; `flush_class_c_tip` **before** body-queue dequeue |
| Not OK | Long-held store locks on IBD/read path, “pause all queries during confirm”, multi appenders |

If a change introduces a new long-held store lock on the IBD/read path, it is the
wrong design — fix the protocol. See `docs/concurrency.md`.

## io_uring: do not flatten custom machines

**Under no circumstances** replace a purpose-built / multi-stage **io_uring
machine** (fused resolve, spend-annotate RMW, pipeline stages, depth-round
machines, etc.) with “simple” batched `pread`/`pwrite` / one-shot
`pread_batch`/`pwrite_batch` submission **without explicit permission from the
user**.

| OK | Not OK without permission |
|----|---------------------------|
| Fix bugs inside the existing machine | Delete/retire a custom machine and call bulk batch helpers instead |
| Thread new flags (e.g. DONTCACHE) through the same SQE path | “Simplify” to serial pread + one big submit for a path that had a staged machine |
| Fall back to pread when uring is unavailable (existing policy) | Rewrite a machine away “because batch is enough” |

If a change seems to require collapsing a machine, **stop and ask** — do not
land the simplification as a drive-by cleanup.

## Create pins: pipeline-local only (no process FIFO)

| Rule | Detail |
|------|--------|
| Pin material | **Plan / batch only** — `batch_pin`, `BatchParents`, plan-local **sparse** `external_parent_outs` (`SparseExternalPin`). SharedParentPin = immutable body compose. No process create pin FIFO |
| IBD confirm intake | **body queue wire only** → lookup → load (no hash-only / Class-A-only confirm) |
| Ancient parents | Cold Class A denserels into plan-local / BatchParents only |
| Header plans | **ConfirmParentCache** always on (MTP / tip-ahead headers) |
| Removed | **CreateResidency**, **OutFifo**, **archive sticky**, half-row / out-slim, **`RBITCOIN_CONFIRM_CACHE`**, **`RBITCOIN_RESIDENCY_BYTES`** |
| IBD sizes | **`conf_plans=`** + body-queue / pipeline meters (no `residency creates=`) |


## GitHub CI must stay green (every commit)

CI is [`.github/workflows/ci.yml`](.github/workflows/ci.yml) (push/PR to
`master`/`main`). **Do not push or leave a commit that would fail the required
`test` job.** A red CI on `master` is incomplete work.

### Required before each code commit (`test` job)

From `nix-shell` (or the same **rustc 1.82** class CI pins):

```bash
cargo fmt --all -- --check          # if dirty: cargo fmt --all
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

| Gate | Expectation |
|------|-------------|
| `cargo fmt --all -- --check` | Clean |
| `clippy … -D warnings` | Clean under `[workspace.lints.clippy]` allows in root `Cargo.toml` |
| `cargo test --workspace` | All non-ignored tests pass |

**Toolchain:** CI pins **rustc 1.82.0** (same class as `nix-shell` / crane). Do not
rely on host “latest stable” alone. Expand clippy allows only for real noise
after a toolchain bump — prefer fixing the code.

### Coverage job

`./scripts/coverage.sh` enforces **100% first-party HTML uncovered-line** (see
`COVERAGE.md`). It runs as a separate CI job (slow). Prefer running it when
touching store/query/consensus hot paths. **Do not land new uncovered production
lines.** The coverage job may be `continue-on-error` while historical gaps are
closed — that is temporary; required `test` must still pass.

If a change cannot pass required gates, **do not commit it as done** — fix, split,
or get explicit user approval for a temporary exception (prefer none).

## Commit + static musl release after code changes

Whenever a turn **changes code** (or you finish a multi-step coding task in that turn):

1. **Pass CI gates** (fmt / clippy / tests — see above). A commit that fails GitHub Actions is incomplete work.
2. **Commit** the working tree with a clear message (what + why). Prefer one commit per logical checkpoint — especially before starting a risky follow-on experiment, so we can roll back. Do **not** leave multi-hour IBD perf/refactor work uncommitted.
3. **Rebuild and install the portable static musl release** so
   `./target/release/rbitcoin-node` matches the tree. This is **mandatory every
   code-changing turn** — not optional after tests.

### Required recipe (only this — single `nix build`)

```bash
nix build .#rbitcoin-musl --out-link result
mkdir -p target/release
install -m 755 result/bin/rbitcoin-node result/bin/rbitcoin-cli target/release/
file target/release/rbitcoin-node   # must say "statically linked" (musl)
```

Musl builds use **crane** (deps derivation + app derivation). After the first
full deps build, **crate-only edits** recompile workspace crates against a
cached `cargoArtifacts` layer — still one `nix build`, not a host `cargo
build --release`.

### Do **not** run for day-to-day agent turns

| Command | When |
|---------|------|
| `./scripts/repro-check.sh` | **Release / digest gate only** — realize + **two** forced `--rebuild`s. Slow by design. Never as the post-edit install step. |
| `./scripts/repro-check.sh both` | Even heavier (musl + glibc). Release only. |

Day-to-day portable install = **one** `nix build .#rbitcoin-musl` (recipe above).
Byte-identity claims for a revision = `./scripts/repro-check.sh` once at release.

### Forbidden for the operator binary

| Do **not** run | Why |
|----------------|-----|
| `nix-shell --run 'cargo build -p rbitcoin-node --release'` | Dynamic **Nix glibc** link; dies off-store with `No such file or directory` |
| `cargo build --release` (host toolchain) | Same class of non-portable binary |
| Leaving `target/release/` as the last **debug** or glibc build | User restarts IBD from that path |

`nix-shell` / `cargo test` for **tests** is fine. Only the **shipped** node/cli
under `target/release/` must come from `nix build .#rbitcoin-musl`.

Skip commit/build only when the turn was pure discussion with no
compile-affecting edits. If you cannot commit (hooks, secrets, user said not
to), still do the static musl install and say the tree was **not** committed.

## Tests required for code changes

- **Always ship test coverage with behavioral code changes.** Prefer unit tests next to the code (`#[cfg(test)]` in the same crate) or focused integration tests in `rbitcoin-test` / crate `tests/`. Pure docs/comments need no tests.
- **Bug fixes must include a regression test** that fails without the fix and passes with it. Do not land a “fix” that only re-describes production logs; encode the failing case (fixture block, synthetic store, prevout/script edge) so it cannot silently come back.
- Run the new/related tests before commit (e.g. `cargo test -p <crate> …` or the scenario that covers the change). If a full-store mainnet case cannot run in this VM (see datadir notes), still add a synthetic/unit regression that pins the logic.

## Simplification / lean-code rules (apply while editing)

| Rule | Detail |
|------|--------|
| **Shared helpers** | Prefer one production implementation (composition or shared fn) over copy-paste probe/hash/layout math across modules. Put the helper in the **lowest crate that owns the concept** (`open_address` for FNV/open-hash, etc.). |
| **Invariants > silent fallbacks** | On confirm/store hot path, if load or body load promised a fact (range present, packed decode, denserels for need_vouts), missing fact → `StoreError::Corrupt("invariant: …")` (or consensus wrap). Do **not** soft-continue to a colder path that hides bugs. Env/protocol multi-path (io_uring off, multi-spender list, RPC reconstruct) stays non-invariant. |
| **No test-only production APIs** | Do not add `*_for_test` / budget overrides / backdoors on production types when tests can use real clamps (large payloads, env, or public constructors). Prefer demoting or deleting over growing `cfg(test)` surface that does not exist for dependent crates. |
| **No re-implemented oracles in tests** | A test must drive the **shipped** function. Local helpers that re-code the unit under test and then “assert” that helper are test theater — delete them. |
| **Collapse same-entry duplicates** | Prefer one unit test next to the shipped path over twin unit+integration suites covering the same lines. Keep the closer entry-point test; drop the other only when coverage remains. |
| **Compile/test lean** | Prefer fewer full-store opens, less fixture copy-paste, and no giant dual test modules for the same slim/filter helper. Measure before claiming wall-time wins. |

## Datadir / store on this workspace (do not open in the agent VM)

The workspace is mounted into the agent VM as **9p** (`workspace` on `/home/agent/workspace`, `trans=virtio`). On this mount:

- Store/mempool tables are **map-free** (pread/pwrite only) — open should work without `MAP_SHARED`.
- Prefer `/tmp` fixtures for agent correctness tests (synthetic stores).

**Perf A/B** is **operator-host only**, with the musl static binary — never agent-VM timings. See [`docs/io-modality.md`](docs/io-modality.md).

### What works instead

- Read **logs** the user leaves in-tree (`signet-ibd.log`, etc.).
- Inspect store files with **pread**/Python struct parsing of HWMs, headers when useful for offline forensics.
- Reproduce with **synthetic fixtures** and `rbitcoin-test` scenarios under `/tmp`.
- Ask the user to run the node / confirm diagnostics / **host musl benches** on their host (normal local FS).

## No dead code warnings silenced unless there is an absolutely bulletproof justification.

Do not leave dead code around. Delete it. Don't silence warnings unless there
is bulletproof justification

Same goes for #[cfg(test)].

## Do test-driven development when practical

Always for bugs, make sure to create a test the replicates the bug, run it to
see it fail, then fix the bug and run the test to see it pass.

For features, ideally we'd write a scenario test that fails without the
required feature before beginning and then implement the feature and see the
test pass.

For performance, ideally we'd have a benchmark before we begin development
that shows a clear change after.

## IBD / process memory leak prevention

**Full rules:** `docs/ibd-memory.md`. Summary for agents:

1. **Distinguish** process-owned heap (Rust structures, confirm pipeline wire,
   in-RAM body queue) from **kernel page cache** under store mmaps (`RssFile`).
   Do not “fix” RSS by gutting intentional caches (body queue, ConfirmParentCache header plans).
2. **Unified path only:** peer → in-RAM **body queue** → confirm lookup/load/
   scripts/commit (sole Class A). **No** dual-track `ArchiveJob` / ContigPark.
   Unknown-height `BlockFramed` → `mark_missing` and re-getdata after height.
   Body queue is **RAM-only** (redownload on restart) to avoid double disk write
   of every block; soft densify assign uses two limits (no hysteresis): under
   ~100 MiB free densify ahead; over ~100 MiB only heights confirm will consume
   in the next ~1 min at tip rate.
3. **Soft budgets are request-limited only.** Always accept already-requested
   block bytes into the body queue (`block_queue_offer` ignores soft assign
   limits), even if that overshoots soft depth. Bound memory by limiting new
   densify **getdata assign** — never by stalling TCP reads or Full-dropping
   bodies already on the wire.
4. **Tests** must tear down intentional caches with **production** APIs (table
   below) — not a secret free-all that masks production leaks.
5. **Regression filters:** body-queue soft depth / presence lifecycle / confirm
   reject paths as listed in `docs/ibd-memory.md`.

### Production clear / evict APIs (tests must call these)

| Structure | Production API |
|-----------|----------------|
| Soft densify depth | Bound **only** via body-queue soft assign (100 MiB free / 1 min confirm window) — never receive-side Full-drop |
| Confirm plans/headers | **`ConfirmParentCache::advance_tip`** (write `post_commit`) |
| Pipeline pins | Drop with plan/batch; **no** process pin FIFO. Tests tear down via production plan drop / batch drop |
| Ordered maps | **`IbdWorkState::hygiene`** |
| Body presence | **`BodyPresence::hygiene_retain`** (rejected retained by design) |

---
> Source: [reardencode/rbitcoin](https://github.com/reardencode/rbitcoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
