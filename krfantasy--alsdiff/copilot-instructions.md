## alsdiff

> MainTrack is a singleton (master output track) with special behavior:

# ALSDiff Agent Instructions

## Critical Patterns (Non-Obvious)

### MainTrack Singleton
MainTrack is a singleton (master output track) with special behavior:
- No `id` field
- `has_same_id _ _ = true` - always returns true
- `id_hash _ = Hashtbl.hash 0` - constant hash
- Never apply regular track ID logic to MainTrack

### PPX Derive System
Core types require specific PPX derives for the diff system:
```ocaml
[@@deriving eq, id, patch]       (* Standard derive for diffable types *)
[@@deriving eq, patch]            (* For types without identity fields *)
[@@patch.generate_diff]           (* Triggers automatic diff generation *)

[@id.id]                          (* Marks identity field for matching *)
[@id.ref]                         (* Marks reference to type with identity *)
[@patch.skip]                     (* Excludes field from patch generation *)
[@patch.identity]                 (* Include in patch but skip validation *)
```

**Never remove `[@@deriving ...]` annotations** - they generate essential code.

### Structured Diff System
Three kinds of changes from `lib/base/diff.ml`:
- `structured_update` - value changes (`Foo.Patch.t structured_update`)
- `structured_change` - option/conditional changes (`(Foo.t, Foo.Patch.t) structured_change`)
- `atomic_patch` - for atomic types (`{ oldval : 'a; newval : 'a }`)

### Module Access Patterns
**Always open base modules in library code:**
```ocaml
open Alsdiff_base
open Alsdiff_base.Diff
```

**Submodule access pattern:**
```ocaml
open Alsdiff_live.Track.Routing      (* Not Alsdiff_live.Routing *)
open Alsdiff_live.Track.Mixer
open Alsdiff_live.Track.MidiTrack
open Alsdiff_live.Track.AudioTrack
```

## Common Mistakes to Avoid

1. **MainTrack is not a regular track** - don't apply ID-based logic to it
2. **Don't skip PPX derives** - record types need `[@@deriving eq, id, patch]`
3. **Don't modify `[@id.id]` fields** without understanding identity matching
4. **Handle `Xml.Xml_error` exceptions** - XML parsing can fail
5. **Don't break singleton pattern** - MainTrack's `has_same_id` always returns true

## Code Style

- Line length: 100 characters
- Format: `dune build @fmt`
- Functions: `snake_case`
- Types: `PascalCase`
- Use functional style, avoid mutable state
- Error pattern: `raise (Xml_error (xml, "descriptive message"))`

## Development Commands

- `dune build` - Build
- `dune runtest` - Run tests
- `dune exec alsdiff` - Run main executable
- `dune build @fmt` - Format code
- `dune describe pp lib/live/foo.ml` - Inspect PPX-expanded code

**Important**: Dune commands are exclusive (they use a global lock file). Never issue multiple `dune build` or `dune runtest` commands concurrently - serialize them to avoid lock conflicts.

## Testing

- Tests in `test/test_*.ml` mirror `lib/live/*.ml` structure
- Use `Test_utils.Utils.resolve_test_data_path` for test data
- Exception pattern: `try ... with Xml.Xml_error _ -> true`
- Regression Testing: After any code changes, run `dune runtest` to verify existing tests still pass before committing.

## Debugging

- Investigation: Verify the root cause affects all cases before implementing fixes. Test with multiple examples, not just the reported scenario.

## Code Migration:

- PPX Deriver Migration: 1) Analyze reference implementations (device.ml, track.ml), 2) Identify types to migrate, 3) Create detailed plan, 4) Implement with tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/krfantasy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/krfantasy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
