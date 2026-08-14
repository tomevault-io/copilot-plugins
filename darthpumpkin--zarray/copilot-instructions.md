## zarray

> This repository is a **n-dimensional array library for Zig** (named `zarray`), analogous to NumPy but with Zig's compile-time paradigm.

# Agent Guidelines for ndarray_zig

This repository is a **n-dimensional array library for Zig** (named `zarray`), analogous to NumPy but with Zig's compile-time paradigm.

## 🚧 Quick Start

**Essential command sequence:**
```bash
# 1. Configure build (macOS only, Accelerate framework)
# Edit build_config.zig to set frameworks/libraries/paths

# 2. Build
zig build

# 3. Run tests (two parallel suites)
zig build test          # Run core library tests
zig build test-test     # Install root tests for IDE tools

# 4. Run benchmarks
zig build bench

# 5. Build/run examples
zig build mlp-example
zig build run-mlp-example
```

## 🔑 macOS Requirement

**This library builds only on macOS** due to dependency on the Accelerate framework (for BLAS/LAPACK).

```zig
// In build.zig
if (target.result.os.tag != .macos) {
    return error.UnsupportedTarget;
}
```

**Fixing platform issues:** The error must be addressed at the build level, not by changing platform detection.

## 🏗 Architecture Overview

### Core API (`src/`)

| File | Purpose |
|------|---------|
| `root.zig` | Module root; exports `NamedArray`, `NamedArrayConst`, `index`, `axis_meta`, `bindings` |
| `named_array.zig` | Main `NamedArray`/`NamedArrayConst` type with all operations |
| `named_index.zig` | Axis-typed index types (`NamedIndex`) |
| `axis_meta.zig` | Optional runtime axis metadata |
| `bindings/` | External library bindings (C-based) |

### Bindings (`src/bindings/`)

Organized as separate modules with **explicit scalar type constraints**:

- `bindings/blas.zig` → BLAS operations (basic linear algebra)
- `bindings/blas_ext.zig` → Extended BLAS (higher-level wrappers)
- `bindings/lapack/lapack.zig` → LAPACK (matrix decomposition)
- `bindings/tblis/tblis.zig` → TBLIS (high-performance matrix multiplication)
- `bindings/gsl/gsl.zig` → GSL (statistical libraries)

## 🎯 Design Principles (ADR-0001)

| Principle | Detail |
|-----------|--------|
| **Strict semantics** | Fail at compile-time if possible; runtime panic if not |
| **Explicit broadcasting** | Never implicit; request explicitly via API |
| **View vs allocation** | Clear split: view ops take no allocator; allocating ops require `allocator` param |
| **No hidden allocation** | Methods without `allocator` arg must not allocate |
| **Compile-time axes** | Axis identity is a compile-time enum, not runtime data |

**Default behavior:** Strict/fail-fast, not convenience-oriented.

## 🧪 Testing Strategy

- **Core tests**: `zig build test` runs `root.zig` tests (refAllDecls)
- **Benchmark tests**: `zig build bench` runs `bench.zig` tests
- Tests run in **parallel** via separate top-level steps
- Use `zig build test-test` to install tests for IDE tools

## 📝 Common Pitfalls

### 1. Non-macOS builds

Attempting to build on Linux/Windows will fail.

**Fix:** Do not try to workaround this by changing target detection. The `UnsupportedTarget` error is intentional.

### 2. Axis misuse

Axes are compile-time enums. Attempting to use `i` where an enum tag for a different axis enum exists will cause compilation errors.

### 3. Implicit vs explicit broadcasting

The API requires explicit broadcasting calls (`broadcastTo`, `broadcastAxis`). There is no "magic shape matching."

### 4. Alias violations

Careful with aliasing when merging/transforming views. Contiguous views can alias; non-contiguous views must be made contiguous first.

### 5. Scalar type support

Each binding module declares supported scalar types. Passing an unsupported type will fail at compile time.

## 🛠 Development Patterns

### Adding a new operation

1. Implement in the appropriate file (`named_array.zig` for core ops, binding file for external libs)
2. Use `test` blocks at module level or function level
3. Add a test case exercising the new behavior
4. Consider strict vs mutable variants naming (`op` vs `opAlloc`)

### Adding a new binding

1. Create new module in `bindings/<library>/<name>.zig`
2. Declare supported scalar types as `const SupportedScalars = _`
3. Expose via `bindings.<library>` in `root.zig`

### Working with backends

- **No automatic dispatch** exists. You select the backend explicitly.
- Each binding has explicit constraints; check scalar type support before use.

## 📖 Documentation

- `docs/glossary.md` → Core terminology (axis, rank, shape, stride, view, etc.)
- `docs/adr/` → Architecture decision records
- `docs/agent-sessions/` → Session logs of past work (useful for context)

## 📁 Project Structure

```
.
├─ build.zig              # Build script (target, optimize options, C files, test/bench/example steps)
├─ build_config.zig       # Configurable build parameters (frameworks, paths)
├─ build_config_template.zig
├─ src/
│  ├─ root.zig            # Module root
│  ├─ named_array.zig     # Core array types
│  ├─ named_index.zig     # Axis-typed index
│  ├─ axis_meta.zig       # Optional axis metadata
│  └─ bindings/
│     ├─ blas.zig
│     ├─ blas_ext.zig
│     ├─ lapack/
│     │  └─ lapack.zig
│     ├─ tblis/
│     │  └─ tblis.zig
│     └─ gsl/
│        └─ gsl.zig
├─ vendor/                # External code (if any)
├─ include/               # C/C++ header paths
└─ zig-out/               # Build output
```

## 🔍 Key Files for Orientation

- `src/root.zig` → Public API surface
- `build.zig` → Build steps, test commands, dependency on C files
- `build_config.zig` → Runtime config (editable)
- `docs/glossary.md` → Terminology reference
- `docs/adr/` → Design rationale

## 💡 Tips

- Use `zig build --help` to see all available steps
- Use `zig build -v` to see compilation commands when debugging
- Core API is **header-only** for consumers; no precompiled libs needed for usage

---
> Source: [DarthPumpkin/zarray](https://github.com/DarthPumpkin/zarray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
