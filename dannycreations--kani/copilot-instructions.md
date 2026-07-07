## kani

> Run commands via `make`, never via `cargo` directly (see Restrictions).

# Kani Development Guide

## Commands

Run commands via `make`, never via `cargo` directly (see Restrictions).

```cmd
make check              # Compile + lint only — fast, no test execution
make test               # Full test suite (unit + integration) — slow
make test my_test_case  # Single test, module, or group — fast
make bench              # All benchmarks — very slow, use sparingly
```

---

## Restrictions

### 1. Do not modify `Cargo.toml` profile sections

The `[profile.*]` tables (e.g., `[profile.release]`, `[profile.dev]`) are locked. Edits affect compiler optimizations globally and can silently break reproducibility.

```toml
# ✅ Allowed — adding a dependency
[dependencies]
serde = { version = "1", features = ["derive"] }

# ❌ Forbidden — touching any profile block
[profile.release]
opt-level = 3
```

---

### 2. Never run `cargo test` directly

`cargo test` bypasses the Makefile's safety harness, which guards against infinite loops and memory leaks. Always use `make test` or `make test <name>`.

```sh
# ✅ Correct
make test
make test my_test_case

# ❌ Forbidden — no loop/leak protection
cargo test
cargo test my_test_case
```

---

### 3. Avoid `unsafe` blocks

`unsafe` is permitted only in two cases: (a) FFI (Foreign Function Interface) boundaries, or (b) verified hot-path optimizations where profiling confirms the gain. Every `unsafe` block must include an inline `// SAFETY:` comment documenting the invariant being upheld and a reference to the audit trail.

```rust
// ✅ Permitted — FFI with documented safety invariant
// SAFETY: `ptr` is guaranteed non-null and valid for `len` bytes
//   by the C caller contract in ffi_contract.md §3.2.
unsafe {
  std::slice::from_raw_parts(ptr, len)
}

// ❌ Forbidden — no justification, no safety comment
unsafe {
  *raw_ptr = 42;
}
```

---

### 4. Declare all dependencies at the top of each file

All `use` statements belong at the file header, not inline within functions, `impl` blocks, or match arms. This keeps the dependency graph visible at a glance and eliminates the need for fully-qualified paths scattered through logic.

```rust
// ✅ Correct — all imports at file top
use std::collections::HashMap;
use serde::{Deserialize, Serialize};

pub fn build_index(items: &[&str]) -> HashMap<&str, usize> {
  items.iter().enumerate().map(|(i, k)| (*k, i)).collect()
}

// ❌ Forbidden — import buried inside logic
pub fn build_index(items: &[&str]) -> std::collections::HashMap<&str, usize> {
  use std::collections::HashMap;  // hidden dependency
  items.iter().enumerate().map(|(i, k)| (*k, i)).collect()
}
```

---
> Source: [dannycreations/Kani](https://github.com/dannycreations/Kani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
