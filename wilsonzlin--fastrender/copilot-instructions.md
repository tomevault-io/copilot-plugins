## fastrender

> This file contains **repo-wide** rules shared by all workstreams.

# FastRender (agent instructions)

This file contains **repo-wide** rules shared by all workstreams.

## Workstreams

Pick one workstream and follow its specific doc. Work can proceed **in parallel** across all workstreams.

### Rendering Engine (correctness & capability)

- **Capability buildout (spec-first primitives)**: `instructions/capability_buildout.md`
- **Pageset page loop (fix pages one-by-one)**: `instructions/pageset_page_loop.md`

### Browser Application (user-facing product)

- **Browser chrome (tabs, navigation, address bar)**: `instructions/browser_chrome.md`
- **Browser responsiveness (performance, not aesthetics)**: `instructions/browser_responsiveness.md`
- **Browser page interaction (forms, focus, scrolling)**: `instructions/browser_interaction.md`

### JavaScript (full browser JS support)

- **JS engine (vm-js core, execution, GC)**: `instructions/js_engine.md`
- **JS DOM bindings (document, element, events)**: `instructions/js_dom.md`
- **JS Web APIs (fetch, URL, timers, storage)**: `instructions/js_web_apis.md`
- **JS HTML integration (script loading, modules, event loop)**: `instructions/js_html_integration.md`

### Quick reference: what each workstream owns

| Workstream | Owns | Does NOT own |
|------------|------|--------------|
| `capability_buildout` | CSS, layout algorithms, paint correctness | Page-specific fixes, browser UI |
| `pageset_page_loop` | Fixing specific pages end-to-end | Generic capability work |
| `browser_chrome` | Tabs, address bar, navigation, shortcuts | Visual design, page interaction |
| `browser_responsiveness` | Frame rate, input latency, scroll perf | Visual design, chrome functionality |
| `browser_interaction` | Forms, focus, selection, scrolling | Chrome UI, JS events |
| `js_engine` | vm-js execution, GC, spec compliance | DOM, Web APIs |
| `js_dom` | Document, Element, Node, events | Web APIs, script loading |
| `js_web_apis` | fetch, URL, timers, storage, crypto | DOM, engine internals |
| `js_html_integration` | Script loading, modules, event loop | DOM APIs, engine internals |

## Non-negotiables

- **No page-specific hacks** (no hostname/selector special-cases, no magic numbers for one site).
- **No deviating-spec behavior** as a "compat shortcut". Implement the spec behavior the page depends on (incomplete is OK; wrong is not).
- **No post-layout pixel nudging**; keep the pipeline staged (parse → style → box tree → layout → paint).
- **No panics** in production code. Return errors cleanly and bound work.
- **Keep Taffy vendored** (`vendor/taffy/`) and only use it for flex/grid; do not update it via Cargo.
- **JavaScript execution must be bounded**: the JS engine must support interrupts/timeouts and avoid unbounded host allocations.

## Philosophy & culture

**Read [`docs/philosophy.md`](docs/philosophy.md)** for hard-won lessons, mindset principles, and development wisdom.

**Read [`docs/triage.md`](docs/triage.md)** for priority order, failure classification, and the operating model.

Key points:
- **Correct pixels are the product.** Everything else exists to help us ship correct pixels faster.
- **90/10 rule**: 90% accuracy + capability, 10% performance + infra. This is the GOAL.
- **Data-driven method**: Inject, trace, collect, understand, systematize. This is the HOW.
- **Priority order**: Panics → Timeouts → Accuracy failures → Hotspots → Polish → Spec expansion.
- **No vanity work**: Changes that don't improve pageset accuracy, eliminate crashes, or reduce uncertainty for imminent fixes are not acceptable.
- **Ruthless triage**: If you can't turn a symptom into a task with a measurable outcome quickly, stop and split the work.

## What counts

A change counts if it lands at least one of:

- **New capability** (feature/algorithm implemented) with a regression.
- **Bugfix** (behavior corrected) with a regression.
- **Stability** (crash/panic eliminated) with a regression.
- **Termination** (timeout/loop eliminated) with a regression.
- **Conformance** (meaningful new WPT/fixture coverage).
- **UX improvement** (user-visible quality, responsiveness, visual polish).

### What does NOT count (guard against drift)

- **Tooling/infra-only work** unless immediately used to ship an accuracy/capability/stability win.
- **Perf-only work** unless it fixes a timeout/loop or makes an accuracy fix feasible.
- **Docs-only work** unless it removes confusion actively blocking fixes.

If you find yourself "improving the harness" without changing renderer behavior, **stop and implement the missing behavior**.

## System resources (RAM / time / disk) — mandatory safety

### The cardinal rule: assume everything can misbehave

FastRender processes hostile inputs (arbitrary web pages, user content, fuzzed data). **Any code path can go pathological**: infinite loops, exponential blowups, memory explosions, deadlocks, livelocks, signal-ignoring hangs.

This isn't paranoia — it's operational reality. A test with a subtle bug can spin forever. A layout algorithm can hit a degenerate case. A network request can hang on a broken server. Code under development is especially suspect.

**Every command you run must have hard external limits that cannot be bypassed by the code being run:**
- **Time limits**: `timeout -k` (not just `timeout` — the `-k` sends SIGKILL after a grace period because misbehaving code can ignore SIGTERM)
- **Memory limits**: `run_limited.sh` / `prlimit` / cgroups (misbehaving code can't allocate its way out)
- **Scope limits**: never run unbounded test suites or builds

If a process exceeds limits, treat it as a bug to investigate — not a limit to raise.

### Running renderer binaries (always cap memory)

For anything that executes the renderer (pages, fixtures, benches, fuzz, etc.), run under OS caps:

```bash
timeout -k 10 600 bash scripts/run_limited.sh --as 64G -- \
  bash scripts/cargo_agent.sh run --release --bin fetch_and_render -- <args...>
```

Prefer in-process guardrails when a CLI supports them (e.g. `--mem-limit-mb`, stage budgets), but **OS caps are still required**.

See: `docs/resource-limits.md`

### Cargo builds/tests — NEVER run unscoped

**This is critical. Violating these rules spawns hundreds of rustc/linker processes, exhausts RAM, and kills the host.**

**FORBIDDEN (no exceptions):**
- `cargo build` / `cargo test` / `cargo check` without wrapper scripts
- `cargo test` without `-p <crate>`, `--test <name>`, or `--lib`
- `cargo test --all-features` or `cargo check --all-features --tests`
- `cargo build --all-targets` or `cargo test --all-targets`
- ANY command that compiles all 100+ targets

**MANDATORY:**
- **Always use `bash scripts/cargo_agent.sh`** for all cargo commands
- For builds/tests of the **vendored `vendor/ecma-rs/` workspace**, use:
  - `bash vendor/ecma-rs/scripts/cargo_agent.sh ...` (from repo root), or
  - `bash scripts/cargo_agent.sh ...` (from within `vendor/ecma-rs/`)
  so `cargo` runs against the correct nested workspace and picks up the pinned
  `vendor/ecma-rs/rust-toolchain.toml`.
- **Always scope test runs**: `-p <crate>`, `--test <name>`, `--lib`, or `--bin <name>`
  - Note: `scripts/cargo_agent.sh test` also caps `RUST_TEST_THREADS` on very large hosts to avoid
    spawning hundreds of concurrent test threads. Override with `FASTR_RUST_TEST_THREADS` /
    `RUST_TEST_THREADS` if needed.
- **Always use a timeout for test commands** — tests can hang indefinitely (infinite loops, livelocks,
  waiting on resources). Wrap with `timeout -k` to prevent stuck agents:

```bash
# CORRECT — always use `timeout -k <grace> <limit>`:
#   -k 10 = send SIGKILL 10s after SIGTERM if process ignores SIGTERM
timeout -k 10 600 bash scripts/cargo_agent.sh build --release
timeout -k 10 600 bash scripts/cargo_agent.sh test --quiet --lib
timeout -k 10 600 bash scripts/cargo_agent.sh test --test integration
timeout -k 10 600 bash scripts/cargo_agent.sh test --test allocation_failure
timeout -k 10 600 bash scripts/cargo_agent.sh check -p fastrender

# WRONG — no SIGKILL fallback (process can ignore SIGTERM forever):
timeout 600 bash scripts/cargo_agent.sh test --quiet --lib

# WRONG — CAN HANG FOREVER:
bash scripts/cargo_agent.sh test --quiet --lib
bash scripts/cargo_agent.sh test --test integration

# WRONG — WILL DESTROY HOST:
cargo test
cargo build --all-targets
cargo check --all-features --tests
```

**Timeout guidelines:**
- **Always use `-k 10`** (SIGKILL after 10s grace period) — pathological code can ignore SIGTERM
- Default: `timeout -k 10 600` (10 minutes) for most test/build commands
- For single focused tests: `timeout -k 10 120` (2 minutes) is usually sufficient
- For full harness runs: `timeout -k 10 900` (15 minutes) if needed
- If a test times out, **do not retry indefinitely** — investigate why it's slow/hanging

### Listing tests (avoid broken pipes)

Rust's test harness treats a closed stdout pipe as an error, so commands like:

```bash
bash scripts/cargo_agent.sh test -p fastrender --lib -- --list | head
```

will often fail with `Broken pipe`. Prefer filtering without truncating the stream:

```bash
# OK (reads the full list, then filters)
bash scripts/cargo_agent.sh test -p fastrender --lib -- --list | rg '^animation::'
```

or redirect to a file and inspect it.

If you run unscoped cargo commands, you will compile 100+ binaries with LTO, spawn hundreds of parallel rustc/mold processes, exhaust all RAM, and render the machine unusable. **There are no exceptions.**

### Disk hygiene (`target/`)

`target/` grows without bound. Before loops, check size and clean when over budget:

```bash
TARGET_MAX_GB="${TARGET_MAX_GB:-400}"
TARGET_MAX_BYTES=$((TARGET_MAX_GB * 1024 * 1024 * 1024))
du -xsh target 2>/dev/null || true
if [[ -d target ]]; then
  size_kib="$(du -sk target 2>/dev/null | cut -f1 || echo 0)"
  size_bytes=$((size_kib * 1024))
  if [[ "${size_bytes}" -ge "${TARGET_MAX_BYTES}" ]]; then
    echo "target/ exceeds ${TARGET_MAX_GB}GB budget; running cargo clean (via scripts/cargo_agent.sh)..." >&2
    bash scripts/cargo_agent.sh clean
    du -xsh target 2>/dev/null || true
  fi
fi
```

### Git hygiene (file modes / dirty submodules)

If your clone shows lots of noise diffs like `old mode 100755/new mode 100644` (or submodules showing
`Subproject commit ...-dirty`), **do not push** those changes.

If you have **no intended local changes**, the quickest cleanup is:

```bash
git restore --source=HEAD --staged --worktree .
git submodule foreach --recursive 'git reset --hard'
```

After cleanup, `git status --porcelain` should be clean (or only show your intended diffs).

To proactively catch accidental filemode-only diffs before committing, you can run:

```bash
bash scripts/check_no_filemode_only_changes.sh
```

To automatically revert filemode-only diffs (without touching content changes), run:

```bash
bash scripts/revert_filemode_only_changes.sh
```

If your environment frequently flips the executable bit, consider setting (local-only):

```bash
git config core.filemode false
```

## Regression philosophy (required)

Live pages motivate fixes, but regressions keep them fixed:

- Prefer **unit tests** in `src/` for parsing/cascade/value computation and internal algorithms
  (including paint/backdrop rendering regressions under `src/paint/tests/`).
- Use **integration tests** in `tests/` only for public API coverage and data-driven runners (fixtures, WPT).
- Use an **offline fixture** only when necessary to reproduce real-world interactions end-to-end.

When uncertain, add the regression first, then implement the fix.

## Test organization (mandatory)

### Unit tests: `src/`

Unit tests live in `src/` alongside the code they test:

```rust
// src/layout/flex.rs
pub fn compute_flex(...) { /* ... */ }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_flex_wrap() { /* ... */ }
}
```

Run with: `cargo test --lib`

### Integration tests: `tests/`

Integration tests in `tests/` are ONLY for:
- Public API tests (testing `FastRender` as an external consumer would)
- Data-driven runners (fixtures, WPT)
- The allocation-failure harness (custom allocator)

If you find yourself importing internal modules in `tests/`, stop — that test belongs in `src/`.

There are exactly 2 integration test binaries:
- `tests/integration.rs` — all normal integration tests
- `tests/allocation_failure.rs` — custom allocator tests

**NEVER create new `tests/*.rs` files.** Every extra file becomes a separate test binary and will be rejected.

**NEVER use `#[path = ...]` shims.** Run individual tests using the standard test filter instead.

Run with:
- `cargo test --test integration`
- `cargo test --test allocation_failure`

Run a single test via filter:
- `cargo test --lib <filter>`
- `cargo test --test integration <filter>`

(See "Cargo builds/tests" above for the required `scripts/cargo_agent.sh` wrapper + `timeout -k`.)

## Reference documentation

Non-workstream docs live in `docs/`, not `instructions/`:

- ecma-rs ownership principle: [`docs/ecma_rs_ownership.md`](docs/ecma_rs_ownership.md)
- Test architecture: [`docs/test_architecture.md`](docs/test_architecture.md)
- WebIDL stack: [`docs/webidl_stack.md`](docs/webidl_stack.md)

---
> Source: [wilsonzlin/fastrender](https://github.com/wilsonzlin/fastrender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
