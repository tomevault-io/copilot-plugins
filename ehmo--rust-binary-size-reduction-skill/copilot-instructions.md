## rust-binary-size-reduction-skill

> Reduce Rust binary size safely across CLIs, servers, libraries, WASM targets, and embedded systems. Use when asked to shrink, strip, slim, optimize, or audit Rust build artifacts, Cargo profiles, dependency trees, monomorphization, or post-build packing.


# Shrink Rust Binary

Deterministic and idempotent Rust binary size reduction. Every change is
measured, reversible, and explained. No change is applied blindly.

## Principles

- **Measure first, change second.** Every optimization is bracketed by a size measurement.
- **Idempotent.** Running this skill twice produces the same result. Settings are set to exact values, not toggled.
- **Deterministic.** No randomness, no heuristics that vary between runs. Same codebase = same output.
- **Correctness over size.** Never break functionality. Flag behavioral changes explicitly.
- **Layered.** Techniques are grouped into tiers: Safe (no behavior change), Behavioral (changes panic/debug behavior), Nightly (requires nightly toolchain), and Structural (code changes).

---

## Step 0: Reconnaissance

Before changing anything, gather the full picture.

1. **Find Cargo.toml files:**
   ```bash
   find . -name Cargo.toml -not -path '*/target/*' | head -20
   ```
   Identify the workspace root vs member crates.

2. **Read the root `Cargo.toml`** and any `[profile.release]` section. Record every existing setting.

3. **Read `.cargo/config.toml`** if it exists. Check for existing RUSTFLAGS, linker settings, build-std config.

4. **Check toolchain:**
   ```bash
   rustc --version && cargo --version
   rustup show active-toolchain
   ```
   Record whether stable or nightly is active. This determines which tiers are available.

5. **Measure baseline binary size and compile time:**
   ```bash
   cargo clean --release 2>/dev/null
   time cargo build --release 2>&1 | tail -5
   ```
   Then for each binary target:
   ```bash
   ls -la target/release/<binary-name> | awk '{print $5, $9}'
   ```
   Record the **exact byte count** as the baseline and the **compile time** in seconds. LTO and codegen-units=1 significantly increase compile time (2-10x), so users need this to evaluate the trade-off.

6. **Check for existing strip/debug settings** that may already be applied. Read the full `[profile.release]` block.

7. **Check workspace member overrides:** If this is a workspace, check member crates' `Cargo.toml` files for `[profile.release]` sections that may override workspace root settings. Profile settings in member crates take precedence.

8. **Classify existing optimization level:**
   - **None:** No `[profile.release]` section or only default values → full optimization potential
   - **Partial:** Some settings present (e.g., `strip = true` but no LTO) → moderate potential
   - **Well-optimized:** Already has strip + LTO + codegen-units=1 → only opt-level and behavioral changes remain

   This classification determines which tiers will produce meaningful gains.

9. **Present the reconnaissance report:**
   ```
   ## Baseline Report
   - Toolchain: <stable/nightly version>
   - Binary: <name> = <N> bytes (<human readable>)
   - Existing profile.release settings: <list or "none">
   - Optimization level: <none/partial/well-optimized>
   - .cargo/config.toml: <exists/absent, relevant settings>
   - Workspace: <yes/no, N members>
   - Member profile overrides: <list or "none">
   ```

---

## Step 1: Safe Profile Settings (Tier 1 — No Behavior Change)

These settings affect only optimization strategy and debug metadata. They do not change runtime behavior.

### 1a. Strip debug symbols

**What:** Removes debug symbols and symbol tables from the binary. Does NOT affect runtime behavior. Cargo >= 1.59.

**Why:** Pre-compiled libstd ships with ~4 MB of DWARF debug symbols that get linked into every binary. Even with `debug = false`, libstd symbols persist unless explicitly stripped. As of Rust 1.77+, `strip = "debuginfo"` is the default for release when no debuginfo is requested anywhere, but many projects still pin older toolchains or have custom profiles.

**Setting:**
```toml
[profile.release]
strip = true
```

`strip = true` is equivalent to `strip = "symbols"` — removes both debuginfo AND symbol names. Use `strip = "debuginfo"` if you need symbol names for profiling/backtraces.

**Trade-off:** Backtraces in release builds will show only addresses, not function names or line numbers. For production binaries where panics are caught upstream, this is acceptable. For CLIs where users report panics, consider `strip = "debuginfo"` instead.

**Expected savings:** 30-90% for small binaries (libstd debuginfo dominates); 5-15% for large binaries.

After applying, rebuild and measure:
```bash
cargo build --release 2>&1 | tail -3
ls -la target/release/<binary> | awk '{print $5, $9}'
```

### 1b. Enable Link-Time Optimization (LTO)

**What:** Allows LLVM to optimize across crate boundaries at link time. Removes dead code that per-crate compilation cannot detect. Stable since Rust 1.0.

**Why:** Without LTO, each crate is optimized in isolation. Functions pulled from dependencies but never called survive in the binary. LTO sees the whole program and eliminates them.

**Setting:**
```toml
[profile.release]
lto = true
```

`lto = true` is equivalent to `lto = "fat"` — full cross-crate optimization. `lto = "thin"` is faster to compile but produces slightly less size reduction. Default to `true` for maximum reduction.

**Trade-off:** Significantly increases link time (2x-10x). CI builds get slower. Use `lto = "thin"` if compile time is critical.

**Expected savings:** 10-30% on top of stripping.

### 1c. Reduce codegen units to 1

**What:** Forces the compiler to process the entire crate as a single unit, enabling maximum intra-crate optimization. Default is 16 for release.

**Why:** With 16 codegen units, the optimizer only sees 1/16th of the crate at a time. With 1 unit, it can inline, deduplicate, and eliminate dead code across the entire crate.

**Setting:**
```toml
[profile.release]
codegen-units = 1
```

**Trade-off:** Compilation is single-threaded per-crate, so wall-clock compile time increases. The effect is multiplicative with LTO — both together produce the best results.

**Expected savings:** 1-5% on top of LTO (they share some of the same optimizations).

### 1d. Optimize for size

**What:** Tells LLVM to prefer smaller code over faster code. `"s"` optimizes for size; `"z"` additionally disables loop vectorization.

**Why:** Default `opt-level = 3` aggressively inlines and unrolls loops, which increases code size. Size-optimized levels avoid these expansions.

**Setting:**
```toml
[profile.release]
opt-level = "z"
```

**Default to `"z"`.** Empirical testing across real-world Rust projects (terminal multiplexers, HTTP servers, CLI tools, data format libraries) shows `"z"` produces 12-21% smaller binaries than `"s"` in typical applications. The Cargo docs note results can vary, but `"z"` wins in the overwhelming majority of cases.

**When to try `"s"` instead:** Only if the binary is compute-heavy (crypto, compression, scientific computing) where disabling loop vectorization causes measurable performance regressions. In that case, build with both and keep whichever is smaller.

**Expected savings:** 10-25% on top of other Tier 1 settings. Larger gains on code with many loops and generic-heavy call chains.

### Tier 1 Combined Settings

After evaluating opt-level, the final Tier 1 block should look like:

```toml
[profile.release]
strip = true            # Remove all symbols
opt-level = "z"         # Optimize for size (default; try "s" only for compute-heavy code)
lto = true              # Full link-time optimization
codegen-units = 1       # Single codegen unit for maximum optimization
```

**Note on `debug = false`:** The release profile already defaults to `debug = 0`, and `strip = true` removes any debuginfo from the binary. Explicitly setting `debug = false` has **zero additional effect** on binary size. Only add it if the project explicitly sets `debug = 1` or `debug = "line-tables-only"` in its release profile — in that case, removing it saves compile time (no debuginfo generation).

**Check for existing settings first.** If the project already has `strip = true`, `lto = true`, etc., skip those and only add what's missing. Note what was already present in the report — do not claim savings for pre-existing optimizations.

**Rebuild and measure after applying ALL Tier 1 settings together.** Report:
```
## Tier 1 Results
- Baseline: <N> bytes
- After Tier 1: <N> bytes (<X>% reduction)
- Settings applied: <list only NEW settings, note pre-existing ones>
- Pre-existing: <list settings that were already in place>
- Compile time: <time in seconds>
```

---

## Step 2: Behavioral Changes (Tier 2 — Changes Runtime Behavior)

These settings remove functionality that may or may not be needed. Present each to the user with its trade-off.

### 2a. Abort on panic

**What:** Replaces stack unwinding on panic with immediate process abort. Stable since Rust 1.10.

**Why:** Panic unwinding requires landing pads, personality functions, and the unwinding runtime in every function that could panic. Aborting eliminates all of this.

**Setting:**
```toml
[profile.release]
panic = "abort"
```

**Trade-offs:**
- `catch_unwind` no longer works — panics kill the process immediately
- Destructors (Drop impls) do NOT run on panic — resources may leak
- No backtrace on panic (combined with strip, you get nothing)
- Libraries that depend on unwinding for cleanup will misbehave

**When safe:** CLIs, short-lived processes, microservices behind a process supervisor, WASM targets.
**When dangerous:** Long-running servers managing stateful resources, anything using `catch_unwind` for error recovery.

**Ask the user:** "Apply `panic = "abort"`? This removes stack unwinding on panic. Destructors won't run on panic paths. Safe for CLIs and supervised services. [Y/n]"

**Expected savings:** 5-10%.

### 2b. Overflow checks (usually no-op)

**Note:** `overflow-checks = false` is already the **default** for the release profile. Empirical testing on multiple binaries confirmed **zero additional savings** from explicitly setting it. Only mention this if the project has `overflow-checks = true` set explicitly — in that case, removing it saves a small amount (0-3%).

### Tier 2 Combined Settings

```toml
[profile.release]
strip = true
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"         # Abort instead of unwind on panic
```

Rebuild, measure, report delta from Tier 1.

---

## Step 3: Dependency Audit (Tier 3 — Structural)

### 3a. Install and run cargo-bloat

```bash
cargo install cargo-bloat 2>/dev/null
cargo bloat --release --crates -n 20
```

This shows which crates contribute most to `.text` section size. Present the top 10 to the user.

### 3b. Check for duplicate dependency versions

**This is one of the highest-impact checks.** Many projects unknowingly compile multiple versions of the same crate because different transitive dependencies pin different major versions.

```bash
cargo tree --duplicates 2>&1 | head -40
```

Common duplicates to look for:
- **HTTP clients** (reqwest v0.11 + v0.12 + v0.13): each pulls its own TLS stack
- **Crypto backends** (ring + aws-lc-rs): different deps pull different backends, both get compiled
- **Random number generators** (rand v0.7 + v0.8): older crates pin old versions
- **Error handling** (thiserror v1 + v2, anyhow versions)

**For each duplicate:** check if the newer version can replace both. If not, the fix is upstream (file an issue or patch via `[patch]` in Cargo.toml).

### 3c. Check for crypto backend duplication

A specific high-impact pattern: projects often end up with BOTH `ring` (~150 KB) and `aws-lc-rs` (~670 KB) because different TLS configurations pull different backends.

```bash
cargo tree -i ring 2>/dev/null | head -10
cargo tree -i aws-lc-sys 2>/dev/null | head -10
```

If both appear, consolidate by choosing one backend and configuring all TLS deps to use it:
```toml
# Force ring backend (smaller, pure Rust):
reqwest = { version = "0.13", default-features = false, features = ["rustls-tls-manual-roots"] }
```

### 3d. Identify replacement candidates

For each of the top 5 crates by size, check:

1. **Is it using default features?** Systematically audit with:
   ```bash
   cargo tree --edges features -p <crate-name> 2>&1 | head -20
   ```
   This shows exactly which features are active and why. Much more reliable than grepping Cargo.toml.

2. **Common heavy -> light replacements:**
   - `reqwest` -> `ureq` (saves 200-400 KB; no async runtime needed for sync HTTP)
   - `openssl` / `native-tls` -> `rustls` (saves 4-6 MB of C library; pure Rust)
   - `clap` (derive) -> `clap` with `default-features = false` or `lexopt`/`pico-args`
   - `regex` -> `regex-lite` (if full Unicode support not needed)
   - `serde` + `serde_json` -> `nanoserde` or `miniserde` (for simple cases)
   - `chrono` -> `time` (often smaller); also add `default-features = false` to chrono
   - `tokio` (full) -> `tokio` with minimal features, or `smol`/`async-std`
   - `hyper` -> `tiny_http` (for simple HTTP servers)
   - `log` + heavy backend -> `log` + `env_logger` with minimal features
   - `backtrace` -> `std::backtrace::Backtrace` (stable since Rust 1.65, saves ~120 KB)

3. **Feature flag audit:**
   ```bash
   cargo install cargo-unused-features 2>/dev/null
   cargo unused-features analyze
   cargo unused-features report
   ```
   This identifies feature flags that are enabled but not used.

### 3e. Check for monomorphization bloat

Run cargo-llvm-lines to find heavily monomorphized generics:
```bash
cargo install cargo-llvm-lines 2>/dev/null
cargo llvm-lines --release 2>&1 | head -30
```

If any generic function appears with many instantiations (>5), consider:
- **Outline pattern:** Extract the non-generic body into an inner function. The generic wrapper only converts arguments.
  ```rust
  // Before: fully generic, monomorphized N times
  pub fn process<T: AsRef<Path>>(path: T) { /* 200 lines */ }

  // After: thin generic wrapper + single concrete implementation
  pub fn process<T: AsRef<Path>>(path: T) {
      process_inner(path.as_ref())
  }
  fn process_inner(path: &Path) { /* 200 lines */ }
  ```
- **Trait objects:** Replace `impl Trait` with `dyn Trait` in non-hot paths. One vtable indirection per call vs N copies of the function.
  ```rust
  // Before: new copy for each Read implementation
  fn deserialize<R: Read>(reader: R) { ... }

  // After: single implementation, dynamic dispatch
  fn deserialize(reader: &mut dyn Read) { ... }
  ```
- **`#[inline(always)]` audit:** Grep for `#[inline(always)]`. Each forces duplication at every call site. Remove from functions >20 lines unless benchmarks prove the inline is critical.
  ```bash
  grep -rn 'inline(always)' src/
  ```

### 3f. Report dependency findings

```
## Dependency Audit
- Top 5 crates by size: <list with sizes>
- Duplicate crate versions: <list>
- Crypto backend duplication: <ring/aws-lc-rs/both>
- Feature flag savings available: <list>
- Monomorphization hotspots: <list>
- Recommended replacements: <list>
```

Ask the user which changes to apply.

---

## Step 4: Nightly-Only Techniques (Tier 4)

**Only proceed if the user is on nightly or willing to switch.** Ask first.

### 4a. Build std from source with build-std

**What:** Recompiles libstd from source with your profile settings, allowing LTO, size optimization, and dead code elimination to apply to the standard library.

**Why:** The pre-compiled libstd is built with `opt-level = 3` (speed, not size) and includes the full library. Building from source lets your LTO remove unused parts.

**Prerequisites:**
```bash
rustup toolchain install nightly
rustup component add rust-src --toolchain nightly
```

**Find the host target triple:**
```bash
rustc -vV | grep host | awk '{print $2}'
```

**Build command:**
```bash
cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target <host-triple>
```

Note: Binary will be at `target/<host-triple>/release/<binary>` instead of `target/release/<binary>`.

**What `optimize_for_size` does:** This is a libstd feature flag (not an LLVM flag) that tells the standard library to use size-optimized algorithm variants — smaller formatting internals, simpler hash implementations, etc. It changes *which code paths* libstd uses, complementing `opt-level = "z"` which changes *how LLVM optimizes* those paths.

**Expected savings:** 20-50% on top of Tier 1+2.

### 4b. Share generic instantiations across crates

**What:** Forces the compiler to share monomorphized generic instances across crates instead of each crate getting its own copy.

```bash
RUSTFLAGS="-Zshare-generics=y" cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target <host-triple>
```

**Trade-off:** When combined with full LTO (`lto = true`), the savings are reduced since LTO already deduplicates across crates. Most effective with `lto = "thin"` or no LTO.

**Expected savings:** 5-20% without LTO, 1-5% with LTO.

### 4c. Remove location details

**What:** Strips file/line/column info from panic messages and `#[track_caller]` sites.

```bash
RUSTFLAGS="-Zlocation-detail=none" cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target <host-triple>
```

**Trade-off:** Panic messages become useless for debugging. Acceptable for production binaries with external error reporting.

### 4d. Remove fmt::Debug

**What:** Makes `#[derive(Debug)]` and `{:?}` formatting into no-ops. Removes all derived Debug format strings and functions.

```bash
RUSTFLAGS="-Zlocation-detail=none -Zfmt-debug=none" cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target <host-triple>
```

**Trade-off:** `dbg!()`, `assert!()` error messages, and `unwrap()` error messages become empty. Any code that parses Debug output will break.

### 4e. Immediate abort on panic (no formatting)

**What:** Removes ALL panic formatting machinery. Panics call `abort()` immediately with no string formatting at all.

```bash
RUSTFLAGS="-Zunstable-options -Cpanic=immediate-abort -Zlocation-detail=none -Zfmt-debug=none" \
  cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target <host-triple>
```

Note: We keep `optimize_for_size` since we're optimizing for binary size. The `backtrace` and `panic-unwind` features are already excluded by using `panic_abort` in `build-std`. If you need to also remove the default `backtrace` feature, use `-Z build-std-features=optimize_for_size` (it replaces defaults, not appends).

**Expected total with all Tier 4:** A hello-world drops to ~30 KB on macOS. Real applications see 50-70% reduction from Tier 1 alone.

### Tier 4 Combined Command (maximum reduction)

```bash
HOST=$(rustc -vV | grep host | awk '{print $2}')
RUSTFLAGS="-Zunstable-options -Cpanic=immediate-abort -Zlocation-detail=none -Zfmt-debug=none" \
  cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target "$HOST"
```

Measure the result:
```bash
ls -la target/$HOST/release/<binary> | awk '{print $5, $9}'
```

---

## Step 5: Post-Build Techniques (Tier 5 — Language Agnostic)

### 5a. UPX compression

**What:** UPX creates a self-extracting compressed executable. The binary decompresses itself into memory at startup.

```bash
# Install if needed
brew install upx  # macOS
# apt install upx  # Linux

upx --best --lzma target/release/<binary>
```

**Trade-offs:**
- Adds ~50-100ms startup time for decompression
- Some antivirus software flags UPX-packed binaries as suspicious (malware commonly uses UPX)
- Cannot be combined with code signing on macOS (signature is invalidated)
- Memory usage at startup is briefly 2x (compressed + decompressed)

**When useful:** Distribution where download size matters more than startup time. Container images. Embedded systems with flash size constraints.

**Expected savings:** 50-70% on top of all other optimizations.

**Ask the user:** "Apply UPX compression? Adds ~50-100ms startup latency and may trigger antivirus heuristics. [Y/n]"

### 5b. Debuginfo compression (if keeping debuginfo)

If the binary includes debuginfo (e.g., `debug = "line-tables-only"` for backtraces in production):

**Garbage collect unused debuginfo:**
```bash
llvm-dwarfutil <binary> <binary>-gc
```
Expected savings: 10-20% of debuginfo sections.

**Compress debuginfo sections:**
```bash
objcopy --compress-debug-sections=zlib <binary> <binary>-compressed
```
Expected savings: 60-70% of debuginfo sections. `zstd` compresses ~5% better but has less tool support (e.g., gimli/backtrace in libstd can't decompress zstd).

**Combined (GC then compress):**
```bash
llvm-dwarfutil <binary> <binary>-gc
objcopy --compress-debug-sections=zlib <binary>-gc <binary>-final
```

**Alternative — compress via linker flag:**
```bash
RUSTFLAGS="-Clink-arg=-Wl,--compress-debug-sections=zlib" cargo build --release
```

### 5c. Linker flags for size reduction

The linker can perform size-reducing transformations beyond what the compiler does. These are some of the most impactful post-Tier-2 optimizations.

#### Identical Code Folding (ICF) — 5-20% for generic-heavy code

ICF merges functions with identical machine code. Rust's monomorphization creates many identical copies (e.g., `Option<&T>::unwrap` for different `T` with same size). ICF deduplicates them at link time.

**Linux (requires lld or mold):**
```toml
# .cargo/config.toml
[target.x86_64-unknown-linux-gnu]
rustflags = ["-C", "link-arg=-fuse-ld=lld", "-C", "link-arg=-Wl,--icf=all"]
```

`--icf=all` allows merging even if function addresses differ (safe for most Rust code). Use `--icf=safe` if the code takes function pointers and compares them.

**macOS:** The Apple linker does not support ICF. Use `lld` on macOS for ICF:
```toml
[target.aarch64-apple-darwin]
rustflags = ["-C", "link-arg=-fuse-ld=lld", "-C", "link-arg=-Wl,--icf=all"]
```

#### LLVM Function Merging — 2-10% (complementary to ICF)

LLVM's `-mergefunc` pass merges functions at IR level before codegen. It catches cases ICF misses.

```bash
RUSTFLAGS="-Cllvm-args=-mergefunc-use-aliases" cargo build --release
```

Can be combined with ICF for maximum deduplication.

**IMPORTANT: LTO interaction.** When `lto = true` (fat LTO) is enabled, both ICF and mergefunc have **near-zero effect** because LTO already performs cross-crate deduplication at the LLVM IR level. Empirical testing on real projects (spotify-player, lance-tools) showed 0% additional savings with mergefunc when LTO was active. These techniques are most valuable when:
- LTO is disabled (fast builds)
- LTO is set to `"thin"` (partial optimization)
- Cross-language LTO is not available (C deps remain opaque)

If you already have `lto = true`, skip ICF and mergefunc — they won't help.

#### Garbage Collection of Unused Sections — 1-5%

**Linux (explicit GC — lld does this by default, bfd does NOT):**
```toml
[target.x86_64-unknown-linux-gnu]
rustflags = ["-C", "link-arg=-Wl,--gc-sections"]
```

**macOS (explicit dead stripping):**
```toml
[target.aarch64-apple-darwin]
rustflags = ["-C", "link-arg=-Wl,-dead_strip"]
```

Note: macOS still benefits from explicit `-dead_strip` for size reduction, even though the default Apple linker is fast. However, with `lto = true`, the effect is minimal since LTO already eliminates dead code.

#### Cross-Language LTO — 5-20% (for projects with C dependencies)

If the project links C code (via `cc` crate, `*-sys` crates like `openssl-sys`, `zstd-sys`, `ring`), regular LTO only optimizes Rust code. Cross-language LTO extends optimization to C code too.

```bash
CFLAGS="-flto" RUSTFLAGS="-Clinker-plugin-lto -Clink-arg=-fuse-ld=lld" cargo build --release
```

Requires `lld` and `clang` (not gcc) for the C code.

**Expected savings:** Significant for projects with heavy C dependencies (crypto, compression). Zero for pure Rust projects.

#### Linker selection summary

| Linker | Platform | Speed | ICF | GC-sections | Notes |
|--------|----------|-------|-----|-------------|-------|
| `lld` | Linux/macOS | Fast | Yes (`--icf=all`) | Default on | Best overall for size |
| `mold` | Linux | Fastest | Yes (`--icf=safe/all`) | Default on | Good alternative |
| `bfd` (default) | Linux | Slow | No | **No** (must pass `--gc-sections`) | Avoid for size work |
| Apple ld | macOS | Fast | No | Partial (add `-dead_strip`) | Limited size features |

---

## Step 6: Advanced Code Patterns (Tier 6 — Code Changes)

These require modifying source code. Present as recommendations, not automatic changes.

### 6a. Avoid unnecessary derives

Every `#[derive(Debug, Clone, PartialEq, ...)]` generates code. Audit structs:
```bash
grep -rn '#\[derive(' src/ | head -30
```

- Remove `Debug` from types never printed with `{:?}` in production
- Remove `Clone` from types never cloned
- Remove `PartialEq`/`Eq` from types never compared
- Consider manual impls that delegate to fewer fields

### 6b. Feature-gate heavy functionality

If the binary serves multiple purposes (CLI + library, server + client), gate heavy dependencies behind features:

```toml
[features]
default = ["client"]
server = ["dep:tokio", "dep:hyper"]
client = ["dep:ureq"]
```

### 6c. Use `#[cold]` on error paths

Mark error-handling functions as cold to prevent inlining into hot paths:
```rust
#[cold]
#[inline(never)]
fn handle_error(e: Error) -> ! { ... }
```

### 6d. Audit string formatting

`format!()`, `println!()`, `eprintln!()` pull in `core::fmt` machinery. In size-critical code:
- Replace `format!("{}", x)` with `x.to_string()` where possible
- Use `write!` to a pre-allocated buffer instead of `format!`
- For integer-to-string, consider `itoa` crate (smaller than fmt machinery)

### 6e. Prefer `&str` over `String` in const/static contexts

Static strings don't need heap allocation. Check for patterns like:
```rust
// Wasteful - allocates at runtime
let msg = String::from("hello");
// Better - zero-cost
let msg: &str = "hello";
```

---

## Step 7: Final Report

After all applied tiers, produce a comprehensive report:

```
## Binary Size Optimization Report

### Environment
- Toolchain: <version>
- Target: <triple>
- OS: <os>
- Optimization level at start: <none/partial/well-optimized>
- Pre-existing settings: <list or "none">

### Size Progression
| Stage                  | Size (bytes) | Size (human) | Delta    | % of baseline | Compile time |
|------------------------|-------------|--------------|----------|---------------|-------------|
| Baseline (release)     | <N>         | <X MB>       | -        | 100%          | <Ns>        |
| Tier 1 (safe profile)  | <N>         | <X MB>       | -<X MB>  | <X>%          | <Ns>        |
| Tier 2 (behavioral)    | <N>         | <X MB>       | -<X MB>  | <X>%          | <Ns>        |
| Tier 3 (dep audit)     | <N>         | <X MB>       | -<X MB>  | <X>%          | <Ns>        |
| Tier 4 (nightly)       | <N>         | <X MB>       | -<X MB>  | <X>%          | <Ns>        |
| Tier 5 (post-build)    | <N>         | <X MB>       | -<X MB>  | <X>%          | n/a         |

### Settings Applied (new)
```toml
[profile.release]
strip = true
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"  # if applied
```

### Settings Already Present (pre-existing)
- <list settings that were already in the project's Cargo.toml>

### Dependency Changes
- <list of changes made>
- Duplicate crate versions found: <list>
- Crypto backend: <ring/aws-lc-rs/both/none>

### Code Changes
- <list of changes made>

### Not Applied (and why)
- <list of skipped techniques with reason>

### Recommendations for CI
Add size tracking to CI to prevent regressions:
```yaml
# Example GitHub Actions step
- name: Check binary size
  run: |
    cargo build --release
    SIZE=$(stat -f%z target/release/<binary> 2>/dev/null || stat -c%s target/release/<binary>)
    echo "Binary size: $SIZE bytes"
    if [ "$SIZE" -gt <threshold> ]; then
      echo "::error::Binary size $SIZE exceeds threshold <threshold>"
      exit 1
    fi
```

---

## Reference: Tools

| Tool | Purpose | Install |
|------|---------|---------|
| `cargo-bloat` | Per-crate and per-function size breakdown | `cargo install cargo-bloat` |
| `cargo-llvm-lines` | Count monomorphized generic instantiations | `cargo install cargo-llvm-lines` |
| `cargo-unused-features` | Find unused Cargo feature flags | `cargo install cargo-unused-features` |
| `twiggy` | WASM code size profiler | `cargo install twiggy` |
| `cargo-show-asm` | Inspect assembly output per function | `cargo install cargo-show-asm` |
| `llvm-dwarfutil` | GC unused debuginfo entries | `apt install llvm-<ver>` / from LLVM |
| `upx` | Executable compression | `brew install upx` / `apt install upx` |
| `bloaty` | Google's binary size profiler (multi-lang) | `brew install bloaty` |

## Reference: Typical Savings by Technique

Empirical data from testing across real-world projects (terminal multiplexers, HTTP servers, CLI tools, data libraries, music players):

| Technique | Typical Savings | Notes | Requires |
|-----------|----------------|-------|----------|
| **Tier 1 combined** (strip+lto+cgu1+oz) | **43-68%** from unoptimized baseline | Single biggest win. Projects with existing opts see 10-25% | Stable 1.59+ |
| `strip = true` | 20-50% (largest single setting) | Removes libstd debug symbols + symbol table | Stable 1.59+ |
| `opt-level = "z"` | 10-25% | Beats "s" by 12-21% in most real-world code | Stable 1.28+ |
| `lto = true` | 10-30% | Cross-crate dead code elimination | Stable 1.0+ |
| `codegen-units = 1` | 1-5% | Diminishing returns on top of LTO | Stable 1.0+ |
| **Tier 2: `panic = "abort"`** | **5-14%** on top of Tier 1 | Removes unwinding machinery. Sole contributor to Tier 2 savings | Stable 1.10+ |
| `overflow-checks = false` | **0%** (default in release) | Already disabled in release profile. Only helps if explicitly set to true | Stable |
| **Tier 4: `build-std`** | **20-50%** on top of Tier 1+2 | Recompiles libstd with your settings | Nightly |
| `-Zlocation-detail=none` | 1-5% | Removes file/line from panics | Nightly |
| `-Zfmt-debug=none` | 2-10% | Removes Debug trait formatting | Nightly |
| `panic=immediate-abort` | 5-15% | Removes ALL panic formatting | Nightly |
| Dependency replacement | 5-50% (varies wildly) | Project-specific | - |
| Monomorphization reduction | 5-30% (for generic-heavy code) | Needs cargo-llvm-lines | - |
| UPX compression | 50-70% | Post-build, adds startup latency | External tool |
| Debuginfo GC + compression | 60-70% (of debuginfo sections) | Only if keeping debuginfo | External tools |

### Empirical Results (benchmark across 7 real-world binaries)

All results are **Tier 1+2 combined** (strip + lto + codegen-units=1 + opt-level z + panic=abort):

| Project | Type | Baseline | After Tier 1+2 | Reduction | Compile time delta |
|---------|------|----------|----------------|-----------|-------------------|
| codexmanager-web | HTTP server | 9.0 MB | 3.1 MB | 66% | +14% |
| codexmanager-service | HTTP server | 15.5 MB | 4.9 MB | 68% | — |
| codexmanager-start | CLI launcher | 7.4 MB | 2.7 MB | 63% | — |
| spotify_player | TUI app | 29.6 MB | 7.9 MB | 73% | — |
| zellij* | Terminal mux | 38.0 MB | 27.2 MB | 28% | — |
| lance-tools | Data tool | 2.2 MB | 1.0 MB | 54% | — |
| vector | Observability | 139.3 MB | 36.2 MB | 74% | — |

*zellij baseline already had strip+lto+codegen-units=1; only opt-level z and panic=abort were new

**Median reduction: 66%. Range: 28-74%.** Projects with no existing optimizations see 54-74%. The compile time increase from LTO+codegen-units=1 is modest (~14%) for incremental builds but can be 2-10x for clean builds.

## Reference: Quick Copy-Paste

**Minimum viable (stable, safe — typical 43-68% reduction):**
```toml
[profile.release]
strip = true
opt-level = "z"
lto = true
codegen-units = 1
```

**Aggressive (stable, behavioral change — typical 54-74% reduction):**
```toml
[profile.release]
strip = true
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"
```

**Maximum (nightly):**
```bash
HOST=$(rustc -vV | grep host | awk '{print $2}')
RUSTFLAGS="-Zunstable-options -Cpanic=immediate-abort -Zlocation-detail=none -Zfmt-debug=none" \
  cargo +nightly build --release \
  -Z build-std=std,panic_abort \
  -Z build-std-features="optimize_for_size" \
  --target "$HOST"
```

---
> Source: [ehmo/rust-binary-size-reduction-skill](https://github.com/ehmo/rust-binary-size-reduction-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
