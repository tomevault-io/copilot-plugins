## comfyui-lattice-compositor

> 🚨 CRITICAL: Enforces Straylight Protocol rules for Nix, Bash, and Haskell code. ALL code MUST pass validation before being written. This rule is NON-NEGOTIABLE and ENFORCED on EVERY message.

# Straylight Protocol Enforcement - MANDATORY

## Description
🚨 CRITICAL: Enforces Straylight Protocol rules for Nix, Bash, and Haskell code. ALL code MUST pass validation before being written. This rule is NON-NEGOTIABLE and ENFORCED on EVERY message.

## Globs
**/*.nix, **/*.hs, **/*.sh, **/*.bash, straylight-core/**/*

## Rules

### Before Writing Code
1. Read relevant RFCs:
   - Nix: `straylight-core/docs/rfc/straylight-001-standard-nix.md`
   - Bash: `straylight-core/docs/rfc/straylight-006-safe-bash.md`
   - Scripts: `straylight-core/docs/skills/straylight-script/SKILL.md`

### Nix Rules
- Use lisp-case: `my-thing` not `myThing`
- No `with lib;` → use `inherit (lib) x y;`
- No `rec {` in derivations → use `finalAttrs:`
- No heredocs (`''...''`) → use `writeText` or file imports
- No `if/then/else` in module config → use `mkIf`
- No import from derivation (IFD) → restructure
- No per-flake nixpkgs config → use centralized straylight-naught config
- No camelCase in `straylight.*` namespace → use lisp-case
- Packages MUST have `meta` with `description`, `license`
- Modules MUST have `_class` attribute

### PureScript Rules
- ❌ `undefined` → ERROR (STRAYLIGHT-007) - use Maybe/Option
- ❌ `null` → ERROR (STRAYLIGHT-007) - use Maybe/Option
- ❌ `unsafeCoerce` → ERROR (STRAYLIGHT-007) - use proper type conversions
- ❌ `unsafePartial` → ERROR (STRAYLIGHT-007) - handle all cases explicitly
- ❌ `unsafePerformEffect` → ERROR (STRAYLIGHT-007) - use Effect/Either properly
- ✅ Scripts must `import Straylight.Script` (STRAYLIGHT-004)

### Lean4 Rules
- ❌ `sorry` → complete the proof
- ❌ `axiom` → use theorem with proof
- ❌ `admit` → complete the proof
- ❌ `unsafe` → use safe alternatives

### File Size Rules
- ❌ Files >500 lines → split into smaller modules
- This is CRITICAL for LLM readability

### Literate Programming Rules (STRAYLIGHT-011)
- ✅ Use `.lit`, `.mdx`, `.lit.hs`, `.lit.cpp`, `.lit.purs` extensions
- ✅ Every code block MUST have:
  - `@target`: cpp23, tailwind4, daisyui, radix, vue, or react
  - `@name`: Unique chunk identifier
  - `@description`: Human-readable description
- ❌ Code blocks without annotations → ERROR (STRAYLIGHT-011-E001)
- ❌ Invalid @target values → ERROR (STRAYLIGHT-011-E005)
- ❌ Duplicate chunk names → ERROR (STRAYLIGHT-011-E002)
- ❌ Undefined chunk references → ERROR (STRAYLIGHT-011-E003)

### C++23 Rules
- ❌ `nullptr` without explicit handling → ERROR (STRAYLIGHT-011-E006)
- ❌ Raw `new` / `delete` → ERROR (STRAYLIGHT-011-E007)
- ✅ Use smart pointers (`std::unique_ptr`, `std::shared_ptr`)
- ✅ Use `std::optional` for nullable values

### Bash Rules
- ONLY `exec "$@"` allowed
- NO if/case/for/while → write Haskell script
- NO variables → write Haskell script

### Haskell Rules
- Scripts MUST `import Straylight.Script`
- Use typed tool wrappers from `Straylight.Script.Tools.*`

### Database Rules (Co-Effect Standards)
- ✅ All database operations must satisfy co-effect equations:
  - **Idempotency**: `f(f(x)) = f(x)` - Operations are idempotent
  - **Monotonicity**: `S(x) <= x` - Security level can only increase
  - **Provenance**: All mutations logged - Append-only event log
  - **Associativity**: `(E1 . E2) . E3 = E1 . (E2 . E3)` - Operations compose cleanly
- ✅ Queries ALWAYS return a value (never None) - use ReactiveCell pattern
- ✅ Missing entities auto-create with zero values
- ✅ All mutations trace through co-effect equations
- ✅ Zero-value semantics: Every type has canonical zero value
- See `ARCHITECTURE_IMPLEMENTATION.md` section 2 for details

### Database Access Pattern (Cell Answer)
- ✅ Use ReactiveCell pattern: `cell = ReactiveCell(db, "table", "id")`
- ✅ `await cell.get()` ALWAYS returns value (creates zero if missing)
- ✅ Never return `None` from database queries
- ✅ When a cell in the database is called, it should query back automatically
- ✅ Zero-value semantics: Every type has canonical zero (Text="", Int=0, List=[], etc.)
- See `toolbox/core/db/reactive_cell.py` for implementation

### Type System Standards (System F/Omega)
- ✅ Explicit error throwing - Never return `null` or `undefined`
- ✅ Total functions - Handle all inputs, terminate on all inputs
- ✅ Proof comments required for critical operations:
  - System F/Omega proof comments
  - Type proof comments
  - Mathematical proof comments
- ✅ Rich error messages - Include context, file names, exact values, actionable info
- ✅ No lazy fixes - Zero shortcuts, zero simple fixes
- ✅ No partial functions - All cases must be handled explicitly
- See `LATTICE/docs/QUICK_START_FOR_NEW_CHAT.md` section "System F/Omega" for pattern

### Agent Training Pattern (Engram)
- ✅ Use engram pattern for O(1) lookup agent training data
- ✅ Location: `toolbox/core/memory/engram.py`
- ✅ Pattern: Fast memory retrieval for agent context
- ✅ O(1) lookup performance for agent training examples
- See `ARCHITECTURE_IMPLEMENTATION.md` for architecture details

### File Locations
- Packages: `straylight-core/nix/packages/{name}.hs`
- Scripts: `straylight-core/nix/scripts/{name}.hs`
- Tool wrappers: `straylight-core/nix/scripts/Straylight/Script/Tools/{Name}.hs`

### Validation
ALWAYS validate code before writing. Use MCP tool `straylight_validate` or run `nix run .#lint`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/justinfleek) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
