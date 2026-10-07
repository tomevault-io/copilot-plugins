## refractir

> + Knowledge Background: Optimizing Compilers, Program Analysis, Symbolic Execution, SMT (Bit-Vector Theory)

# RefractIR: Agent Guideline

+ Knowledge Background: Optimizing Compilers, Program Analysis, Symbolic Execution, SMT (Bit-Vector Theory)
+ Implementation Language: C++ 20
+ Primary Source Directory: ./src


## Project Overview

RefractIR (internally called SymIR) is a **CFG-based symbolic intermediate representation** designed for:

- **Program synthesis**
- **Symbolic execution**
- **Constraint generation for SMT solvers (Bit-Vector logic)**

A RefractIR program is a **template**, not a fully concrete program. It may contain **symbols** (unknowns, marked with `?`) whose values are solved later by an SMT solver under constraints derived from:

1. A **specific execution path** through the CFG
2. Explicit **properties** (e.g., `require` and/or other user-provided properties) that must hold on that path

The key design goals are:

- SMT-friendliness (predictable BV constraints, minimal nonlinearity)
- Clear semantics with **strict undefined behavior (UB)**
- Explicit control-flow (CFG, no SSA, user-friendly)
- Simplicity and analyzability over expressiveness

RefractIR deliberately restricts expressions (flat, left-to-right, no parentheses) and uses **LLVM-style syntax** (`@`, `%`, `br`, basic blocks) while remaining language-agnostic.

The current formal specification is **[./docs/SPEC_v0.2.2.md](./docs/SPEC_v0.2.2.md)** (v0.2.2, complete and released — adds function calls (`call`) and interprocedural execution; external declarations (`decl`) with `-I` link resolution and behavioral contracts (`pre`/`post`/`ret`); and standard intrinsics, documented per-intrinsic in [./docs/intrinsics.md](./docs/intrinsics.md)). The v0.2.1 spec (aggregate pointers `ptr [N] T`, `ptr @S`, `ptrindex`, `ptrfield`; SIMD vector types `<N> T` with lane-wise arithmetic and `cmp`; floating-point value model; mask-based `select`) is preserved at [./docs/SPEC_v0.2.1.md](./docs/SPEC_v0.2.1.md), the v0.2.0 pointer baseline at [./docs/SPEC_v0.2.0.md](./docs/SPEC_v0.2.0.md), and the pre-pointer baseline at [./docs/SPEC_v0.1.0.md](./docs/SPEC_v0.1.0.md) for reference.

## Language at a Glance

Key characteristics:

- **Non-SSA**, mutable locals via `let mut`
- **Symbols**: solver-chosen unknowns (`@?x`, `%?y`)
- **CFG-based**: explicit basic blocks (`^label`)
- **Expressions**:
  - Flat, left-to-right
  - `+`, `-` at expression level
  - `*`, `/`, `%` at atom level
- **Division/modulo**: round toward 0 (C / SMT `bvsdiv`, `bvsrem`)
- **select** expression:
  - Lazy (only selected arm evaluated)
  - Expression-level conditional; mask-based `select <N> i1` for per-lane blends (v0.2.1)
- **Floating-point**:
  - `f32` (IEEE 754 single) and `f64` (IEEE 754 double)
  - Finite-only domain: ±∞ and NaN are UB
  - `%` is C's `fmod` (truncated-quotient remainder)
  - All operations use RNE rounding
- **Pointers (v0.2.0)**:
  - `ptr T` type, scalar/pointer pointees
  - `addr lv` produces `ptr T` (requires `let mut` root)
  - `load p`, `store p, v` for read/write through pointers
  - `null` literal (typed by context)
  - Pointer arithmetic: `ptr T ± iN → ptr T`, `ptr T - ptr T → i64` (element distance)
  - No `sym` of pointer type; no pointer/integer casts
- **Aggregate pointers (v0.2.1)**:
  - `ptr [N] T` / `ptr @S` — pointers to arrays and structs
  - `ptrindex <ptr [N] T>, <idx>` → `ptr T` (navigate array element)
  - `ptrfield <ptr @S>, <field>` → `ptr FieldType` (navigate struct field)
  - Packed `sizeof(@S) = Σ sizeof(field_i)`; no padding
  - No aggregate `load`/`store`; navigate to scalar/pointer leaves first
- **Vector types (v0.2.1)**:
  - `<N> T` — fixed-width SIMD vector of scalar elements
  - Lane-wise arithmetic: all scalar operators lift to vectors
  - `cmp <relop> lhs, rhs` — reified comparison → `i1` (scalar) or `<N> i1` (vector mask)
  - Lane access via subscript: `%v[%i]` reads/writes lane `i`
  - Whole-vector copy: `%v = %w;`
  - Not addressable: no `ptr <N> T`, no `addr` on vector locals
  - `sym` of vector type allowed (per-lane independent symbols)
- **Strict UB**:
  - Division/modulo by zero
  - Out-of-bounds array access
  - Reading `undef`
  - Null/uninitialised/out-of-bounds pointer dereference, cross-object pointer arithmetic or relational comparison
  - FP overflow/NaN, float-to-integer out-of-range
  - Vector lane-wise UB, out-of-bounds lane access (v0.2.1)

## Toolchain Overview

| Tool | Role |
|------|------|
| `symirc` | Translate `.sir` to C / WebAssembly |
| `symiri` | Interpret `.sir` programs |
| `symirsolve` | Concretize symbolic programs using SMT |

Documentation of each tool: [./docs/](./docs).

## Compilation / Analysis Pipeline

### Shared Frontend Pipeline

```
Source (.sir)
  ↓
Lexer
  ↓
Parser → AST
  ↓
CFG Builder
  ↓
TypeChecker (BV-aware)
  ↓
Semantic Checker
```

### Tool-Specific Pipelines

`symirc`:
```
Checked AST + CFG
  ↓
Lowering
  ↓
C / WASM Code Generation
```

`symiri`:
```
Checked AST + CFG
  ↓
Symbol Binding (--sym)
  ↓
Interpreter Execution
```

`symirsolve`:
```
Checked AST + CFG
  ↓
Path-based Symbolic Execution
  ↓
SMT Solving
  ↓
Concrete .sir
```

### 1. Lexer
- Converts source text into tokens
- Handles identifiers with sigils (`@`, `%`, `@?`, `%?`, `^`)
- Handles comments and string literals
- No semantic knowledge

### 2. Parser
- Recursive-descent parser
- Builds a **typed, structured AST**
- Preserves source spans for diagnostics
- AST is analysis-oriented (not syntax-oriented)

### 3. CFG Builder
- Indexes basic blocks by label
- Builds successor and predecessor lists
- Validates `br` targets
- Computes traversal orders (e.g., reverse postorder)
- Forms the backbone for all dataflow analyses

### 4. TypeChecker (BV-aware)
- Maps RefractIR integer types to **SMT bit-vectors**
  - `i32` → `(_ BitVec 32)`
  - `i64` → `(_ BitVec 64)`
  - `iN`  → `(_ BitVec N)`
- Ensures:
  - Type correctness of expressions
  - Bitwidth compatibility
  - Correct typing of `select`
  - Assignment compatibility
  - Function return correctness
- Produces **typed annotations** for AST nodes
- Boolean conditions are treated separately from BV integers

### 5. Semantic Checker
Ensures program well-formedness beyond typing:

- Variables and symbols are declared before use
- No duplicate declarations
- Assignment only to `let mut` locals
- Parameters and symbols are immutable
- Definite initialization:
  - Parameters are initialized
  - `undef` is uninitialized
  - Reads before initialization are errors
- CFG consistency checks

### 6. Symbolic Execution / Constraint Generation
(Not fully implemented yet)

- Executes along a **user-selected path**
- Collects:
  - Path conditions from branches
  - Assumptions (`assume`)
  - Required properties (`require`)
- Applies **strict UB pruning**
- Produces BV constraints suitable for SMT solvers

### 7. Language Lower / Translator
- Translate a symbolic or concrete program into an existing language
- First-class support are C and WebAssembly.
- For symbolic program translation, use external function declarations to indicate symbols.
  - C: `extern int func_name_symbol_name(...);`
  - WASM: `import func_name symbol_name (func func_name_symbol_name (....))`


## Project Structure

Goto [./README.md](./README.md)


## Testing – TDD Approach (MANDATORY)

ALWAYS follow a strict Test-Driven Development discipline.

### Required workflow

1. Write **five failing tests** that expose the bug or demonstrate the desired behavior
2. Run the tests **one by one** to confirm they fail
3. Implement the fix or feature
4. Run the tests **one by one** to confirm they pass
5. Add the **smallest additional test** that covers edge cases, in the correct test directory

### Never

1. Disable failing tests
2. Modify tests to avoid triggering bugs
3. Add workarounds that bypass the real issue
4. Implement features without a test demonstrating them first

## Dependency Management

### C++
- Dependencies are managed **manually**
- Prefer header-only or standard-library-only solutions
- When introducing a new dependency:
  - Update `README.md`
  - Clearly document installation steps and versions

### Python (if used for tooling)
- Virtual environment: `./venv`
- Activate with:
  ```bash
  source venv/bin/activate
  ````

* Dependencies:
  * `requirements.txt` – runtime
  * `requirements.dev.txt` – development
* Always pin exact versions

## Best Practices

1. Use git frequently and meaningfully
2. Follow **Conventional Commits**
3. Keep `README.md`, `SPEC.md`, `AGENT.md`, and `TODO.md` up to date
4. Fix **all compiler warnings**
5. Keep a clean, layered project structure
6. Write high-quality comments that explain *why*, not *what*


## Floating-point serialization invariant (MANDATORY)

RefractIR carries `f32`/`f64` values bit-exactly across **every** boundary
that involves text — `.sir` source, descriptor JSON, SOLVED/PARAMS/RETURN
headers, model-dump files, and CLI positional args. The invariant is:

> **One canonical bit-exact format. One canonical parser. Used everywhere
> RefractIR text crosses a process or file boundary.**

### The two canonical entry points

- **`refractir::formatDouble(double)`** (`include/ast/ast.hpp`) — emits the
  shortest decimal string that round-trips via `std::to_chars(…,
  std::chars_format::shortest)`, with `.0` appended if neither `.` nor
  exponent is present so int/float dispatch on the resulting string is
  unambiguous and signed zero survives.
- **`refractir::parseFloatLiteral(std::string)`** (`include/ast/ast.hpp`) —
  uses `std::strtod` directly. Subnormals (returned values < `DBL_MIN`)
  are accepted; only true overflow to `±HUGE_VAL` raises. **Never call
  `std::stod`** anywhere in RefractIR — libstdc++ throws `out_of_range` on
  any `ERANGE`, including valid subnormals, and a perfectly representable
  denormal would abort the interpreter.

### Where the invariant must hold

All of these MUST go through the canonical pair:

- `SIRPrinter::printDouble` — `.sir` source emission.
- `rysmith` `fmtModelVal` — descriptor JSON + SOLVED header.
- `symirsolve` `fmtVal` — SOLVED header on solved programs.
- `symirsolve` model-dump JSON output.
- `reify::rewrite` `parseF64` — descriptor JSON read-back.
- Parser/Lexer float tokens — already routed through `parseFloatLiteral`.
- The interpreter's CLI positional-arg parser.

### Where it intentionally diverges

- `src/backend/c_backend.cpp` and `src/backend/wasm_backend.cpp` emit
  literals in **C** and **WAT** grammar respectively (suffixes, infinity
  syntax, etc.), so each backend has its own bit-exact formatter. Each
  carries a comment pointing back to `refractir::formatDouble` to flag the
  divergence as intentional.

### When you add a new producer or consumer

If you are about to write any of these patterns, **stop and use the
canonical pair instead**:

- `std::stod`, `std::stof`, `std::atof`
- `std::cout << double_value` with default precision
- `std::to_string(double)`
- `std::ostringstream` with explicit `precision(17)` or `max_digits10`
- `printf("%f", …)` or `printf("%g", …)` for FP output

`printf("%a", …)` is acceptable for **bit-exact xval output** (the
interpreter's `Result:` line uses this so the C-side `printf("Result:
%a\n", …)` can be compared byte-equal in the diff test). Hex-float form
is parseable by `strtod` and lossless by construction; it just isn't the
canonical *decimal* form.

## Before Starting Work

1. Review recent history:

   ```bash
   git log [--oneline] [--stat] [--name-only] # Show brief/extended history
   git show [--summary] [--stat] [--name-only] <commit> # Show brief/extended history of a commit
   git diff <commit> <commit> # Compare two different commits
   git checkout <commit> # Checkout and inspect all the details of a commit
   ```
2. Understand existing design decisions before changing behavior
3. For large tasks, commit incrementally with clear messages

## Before Saving Changes

ALWAYS:

1. Clear all compiler warnings
2. Format code with `clang-format`
3. Ensure all tests pass (timeouts excepted)
4. Check changes with `git status`
5. Split work into small, reviewable commits
6. Use Conventional Commit messages:

```text
<type>[optional scope]: <title>

<body>

[optional footer]
```

* Title ≤ 50 characters
* Body explains intent and design impact

**Remember:**
RefractIR prioritizes *clarity, analyzability, and solver-friendliness* over surface-level convenience.
Preserve these properties in every change.

---
> Source: [connglli/RefractIR](https://github.com/connglli/RefractIR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
