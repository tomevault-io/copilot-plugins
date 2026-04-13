## ssz-lisp

> SSZ-Lisp is a complete implementation of Ethereum's SSZ (Simple Serialize) format in Coalton, a statically-typed functional language that compiles to Common Lisp. The library provides serialization, deserialization, and merkleization for all SSZ types.

# CLAUDE.md - Project Documentation for AI Assistants

## Project Overview

SSZ-Lisp is a complete implementation of Ethereum's SSZ (Simple Serialize) format in Coalton, a statically-typed functional language that compiles to Common Lisp. The library provides serialization, deserialization, and merkleization for all SSZ types.

## Quick Commands

```bash
# Load the library
sbcl --eval '(ql:quickload "ssz")'

# Run unit tests (181 tests)
sbcl --eval '(ql:quickload "ssz/tests")' --eval '(ssz-tests/runner:run-tests)'

# Run spec tests (1,760 tests)
sbcl --eval '(ql:quickload "ssz/tests")' --eval '(ssz-tests/runner:run-spec-tests)'

# Run benchmarks
sbcl --eval '(ql:quickload "ssz/bench")' --eval '(ssz-bench:run-all-benchmarks)'
```

## Architecture

### File Structure

```
src/
├── package.lisp      # Package definition, all exports
├── constants.lisp    # SSZ constants (+bytes-per-chunk+, etc.)
├── types.lisp        # Core types: Bytes, U128, U256, SSZError
├── serialize.lisp    # Basic type serialization (U8-U256, Boolean)
├── deserialize.lisp  # Basic type deserialization
├── merkle.lisp       # SHA256, merkleization, zero-hash
├── merkle-cache.lisp # Zero hash cache initialization (pure CL)
├── composite.lisp    # SSZVector, SSZList, SSZBitvector, SSZBitlist, SSZUnion
├── typeclass.lisp    # SSZSerialize/SSZDeserialize/SSZHashTreeRoot instances
└── container.lisp    # Container serialization helpers

test/
├── package.lisp      # Defines #:ssz-tests (Coalton) and #:ssz-tests/runner (pure CL)
├── helpers.lisp      # Coalton test helper functions (in #:ssz-tests package)
├── unit-tests.lisp   # Unit test runner (in #:ssz-tests/runner, pure CL, no cl: prefixes)
└── spec-tests.lisp   # Spec test runner (in #:ssz-tests/runner, pure CL, no cl: prefixes)

bench/
└── benchmarks.lisp   # Performance benchmarks
```

### Load Order

Files are loaded serially in the order listed in `ssz.asd`. Dependencies flow downward:
1. `package.lisp` - defines the SSZ package
2. `constants.lisp` - no dependencies
3. `types.lisp` - defines Bytes, U128, U256, SSZError
4. `serialize.lisp` - depends on types
5. `deserialize.lisp` - depends on types, serialize
6. `merkle.lisp` - depends on types, serialize (defines `*zero-hash-cache*` variable)
7. `merkle-cache.lisp` - initializes `*zero-hash-cache*` at load time
8. `composite.lisp` - depends on all above
9. `typeclass.lisp` - defines type class instances
10. `container.lisp` - container helpers

## Coalton-Specific Patterns

### Readtable

All Coalton files must set the readtable:
```lisp
(in-package #:ssz)
(named-readtables:in-readtable coalton:coalton)
```

### Mixing Coalton and Common Lisp

**Within Coalton code**, use `lisp` form to call CL:
```lisp
(define (some-function x)
  (lisp ReturnType (x)
    (cl:some-cl-function x)))
```

**For pure CL code** (like `merkle-cache.lisp`), switch readtable at file start:
```lisp
#.(cl:progn (named-readtables:in-readtable :standard) cl:nil)
(cl:in-package #:ssz)
;; Now write pure CL code with cl: prefixes
```

### Calling Coalton from CL

Coalton functions that take `Unit` require `coalton:Unit`:
```lisp
;; From CL test code:
(ssz:ssz-tests-some-function coalton:Unit)
```

### Type Annotations

Use `the` for explicit type annotations:
```lisp
(let ((vec (the (SSZVector U32) (SSZVector elements))))
  ...)
```

For deserialization results, annotate to help type inference:
```lisp
(match (the (SSZResult (SSZVector U8)) (ssz-deserialize data))
  ((Ok result) ...)
  ((Err e) ...))
```

## Key Design Decisions

### Bytes Type

`Bytes` is an alias for `(vec:Vector U8)` - Coalton's mutable vector type. This provides:
- O(1) random access
- Efficient slicing via `bytes-slice`
- Compatible with Ironclad's SHA256

**Note**: There's a bug in `vec:extend!` with copied vectors. Use `push!` in a loop instead:
```lisp
(define (bytes-append a b)
  (let ((result (vec:copy a)))
    (for elem in (iter:into-iter b)
      (vec:push! elem result))
    result))
```

### U128/U256 Representation

Large integers use native CL arrays for performance:
```lisp
(repr :native (cl:simple-array cl:t (4)))
(define-type U256 "256-bit unsigned integer")
```

Serialization is inlined (no function calls) for performance:
```lisp
(define (serialize-u256 val)
  (let ((result (make-bytes 32)))
    (let ((w0 (u256-word 0 val)) ...)
      ;; 32 individual setf operations
      ...)))
```

### Zero Hash Caching

Zero hashes (used for merkle tree padding) are precomputed at load time:
- `merkle.lisp` declares `*zero-hash-cache*` variable
- `merkle-cache.lisp` initializes it with depths 0-32
- `zero-hash` function does O(1) lookup

This provides ~5x speedup for merkleization.

### Type Classes

Three type classes define the SSZ interface:
- `SSZSerialize` - `ssz-serialize`, `ssz-fixed-size?`, `ssz-fixed-size`
- `SSZDeserialize` - `ssz-deserialize`
- `SSZHashTreeRoot` - `hash-tree-root`

Instances are defined in `typeclass.lisp` for all SSZ types.

### Composite Types

All composite types store their elements as Coalton `List`:
```lisp
(define-type (SSZVector :a)
  (SSZVector (List :a)))

(define-type (SSZList :a)
  (SSZList UFix (List :a)))  ; UFix is max-length
```

### Error Handling

All fallible operations return `(SSZResult :a)` which is `(Result SSZError :a)`:
```lisp
(match (deserialize-u32 data)
  ((Ok val) ...)
  ((Err (DeserializationError msg)) ...))
```

## Testing Architecture

### Two-Package Pattern

Tests use two separate packages to avoid `cl:` prefix verbosity:

1. **`#:ssz-tests`** (Coalton) - Test helper functions in `helpers.lisp`
2. **`#:ssz-tests/runner`** (pure CL) - Test runners in `unit-tests.lisp` and `spec-tests.lisp`

**`#:ssz-tests` package** uses Coalton for test logic:
```lisp
(coalton-toplevel
  (declare ssz-tests-some-test (Unit -> Boolean))
  (define (ssz-tests-some-test _)
    ;; Use ssz: prefix for SSZ symbols
    (let ((data (ssz:serialize-u32 42)))
      ...)))
```

**`#:ssz-tests/runner` package** uses only `#:cl`, so no prefixes needed:
```lisp
(in-package #:ssz-tests/runner)

(defun run-tests ()
  (test-case "Some test"
    (let ((result (ssz-tests:ssz-tests-some-test coalton:Unit)))
      (assert (eq result coalton:True)))))
```

This pattern keeps both the SSZ public API clean and the test code readable.

### Spec Tests

Spec tests use JSON test vectors from `test/spec-test-data.json`. The file is generated by `scripts/extract-spec-tests.py` from the official Ethereum consensus-spec-tests.

## Common Issues

### "Expected type X but got (Result SSZError X)"

Functions like `ssz-bitvector-from-list` and `ssz-bitlist-from-list` return `Result` types. Must pattern match:
```lisp
(match (ssz-bitlist-from-list 100 bits)
  ((Err _) ...)
  ((Ok bl) (serialize-bitlist bl)))
```

### Type Inference Failures

Add explicit type annotations:
```lisp
(the (SSZResult (SSZList U8)) (ssz-deserialize data))
```

### Readtable Issues in Pure CL Code

When writing pure CL in a Coalton-using package, either:
1. Use a separate package that only uses `#:cl` (like `#:ssz-tests/runner`)
2. Or use the readtable switch and `cl:` prefixes (like `merkle-cache.lisp`)

### Deserializing Small Vectors

Vectors/Lists with total data < 4 bytes use `deserialize-vector-fixed-infer` which tries element sizes 1, 2, 4, 8, etc. This was fixed to handle `Vector[U8]` with few elements.

## Performance Notes

Current benchmarks (Apple M1, SBCL 2.5.4):
- Basic serialization (U8-U64): 37-83M ops/sec
- Basic deserialization (U8-U64): 39-73M ops/sec
- SHA256 (32 bytes): 191K ops/sec
- Merkleize (4 chunks): 80K ops/sec
- Container serialize (3 fields): 4.0M ops/sec
- Container deserialize (3 fields): 6.8M ops/sec
- Container hash-tree-root (2 fields): 317K ops/sec

Bottleneck is SHA256 (via Ironclad). Future optimizations could:
- Batch SHA256 operations
- Use memory pooling for intermediate allocations
- Implement lazy merkleization

## Adding New Features

### New Basic Type

1. Add serialize function in `serialize.lisp`
2. Add deserialize function in `deserialize.lisp`
3. Add type class instances in `typeclass.lisp`
4. Export from `package.lisp`
5. Add tests in `test/package.lisp`

### New Composite Type

1. Define type in `composite.lisp`
2. Add serialize/deserialize functions
3. Add hash-tree-root function
4. Add type class instances in `typeclass.lisp`
5. Export from `package.lisp`
6. Add roundtrip and edge case tests

## Reference Projects

Reference implementations are stored in `refs/` (gitignored). Fetch them with:
```bash
./scripts/fetch-refs.sh
```

This shallow-clones:
- `refs/coalton` - [Coalton](https://github.com/coalton-lang/coalton) compiler/language
- `refs/consensus-specs` - [Ethereum consensus specs](https://github.com/ethereum/consensus-specs)
- `refs/ethereum_ssz` - [Rust SSZ](https://github.com/sigp/ethereum_ssz) (Lighthouse)
- `refs/fastssz` - [Go SSZ](https://github.com/ferranbt/fastssz) (high-performance)
- `refs/py-ssz` - [Python SSZ](https://github.com/ethereum/py-ssz) (reference impl)

## References

- [SSZ Specification](https://ethereum.github.io/consensus-specs/ssz/simple-serialize/)
- [Coalton Documentation](https://coalton-lang.github.io/)
- [Consensus Spec Tests](https://github.com/ethereum/consensus-spec-tests)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samdefmacro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/samdefmacro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
