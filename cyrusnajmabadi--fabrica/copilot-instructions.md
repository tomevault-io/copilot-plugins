## fabrica

> How to handle JIT baseline test failures and add new baselines


# JIT Baseline Tests

## What they are

Golden-file tests that capture the native ASM the JIT produces for critical visitor-pattern
methods. They verify that `typeof` dead-branch elimination, struct devirtualization, and
inlining are working as expected. Each baseline is stored at:

```
tests/baselines/jit/{os}-{arch}-net{major}.{minor}.{patch}/{method}.asm       # primary baseline
tests/baselines/jit/{os}-{arch}-net{major}.{minor}.{patch}/{method}.v{N}.asm  # variant (alternative valid encoding)
tests/baselines/jit/{os}-{arch}-net{major}.{minor}.{patch}/{method}.ref       # redirect to another version's .asm
```

A `.ref` file is a plain text file containing a relative path to the actual `.asm` baseline.
Use `.ref` files to deduplicate when a new runtime version produces identical JIT output
to an existing version. This avoids duplicating files and works on all platforms (unlike symlinks).

**Variants** (`.v2.asm`, `.v3.asm`, etc.) represent alternative valid encodings the JIT may
produce non-deterministically — e.g., RIP-relative vs register-indirect addressing depending
on ASLR / memory layout. The test passes if the actual output matches ANY variant.

The platform key includes the .NET runtime version because even patch releases can change
JIT codegen.

## Three layers of verification

The tests validate JIT optimization at three levels:

1. **Runtime correctness**: `Fabrica.JitBaseline` asserts that refcounts are actually updated
   correctly (exit code 1 on failure). This catches functional regressions regardless of ASM.

2. **Structural validation** (`AssertTypeofEliminated`): Before comparing baselines, the test
   performs two independent checks to verify `typeof` elimination:

   - **Negative check**: The ASM must NOT contain `GetTypeFromHandle`, `RuntimeTypeHandle`, or
     `CORINFO_HELP`. These strings only appear when the JIT failed to constant-fold a typeof
     check and fell back to runtime type resolution.

   - **Positive check**: Count the architecture-specific refcount decrement instruction. Each
     fully-inlined `RefCountTable.Decrement` call produces exactly one:
     - ARM64: `sub wN, wN, #1`
     - x64: `dec dword ptr [...]` or `add dword ptr [...], -1`
     
     On x64, wrapper methods are thin `call` stubs (the JIT doesn't inline into `NoInlining`
     methods). The positive check is skipped for these; it runs on x64 callee baselines where
     the actual inlining occurs. The expected count per scenario:

     | Scenario                          | Decrements | Why                                          |
     |-----------------------------------|:----------:|----------------------------------------------|
     | `visit-same-type`                 | 1          | typeof true → direct Decrement               |
     | `visit-different-type`            | 0          | typeof false → entire body eliminated         |
     | `enumerate-decrement-same-type`   | 2          | Both children same type → 2 Decrements        |
     | `enumerate-decrement-mixed-type`  | 1          | Cross-type branch eliminated → 1 Decrement   |
     | `enumerate-decrement-multi-type`  | 2          | Both typeof branches active → 2 Decrements   |

3. **Baseline comparison**: The exact normalized ASM is compared against a golden file. This
   catches any codegen regression, even ones that pass structural checks.

### Wrapper vs. Callee Baselines

On ARM64, the JIT fully inlines callee methods into the `NoInlining` wrappers. The wrapper
baselines contain the complete proof of typeof elimination.

On x64, the JIT doesn't inline into `NoInlining` wrappers. The wrapper baselines show thin
`call` stubs. Separate **callee baselines** capture the standalone callee methods
(`EnumerateChildren`) where typeof elimination actually occurs. Callee tests use
`JitOutput_CalleeMatchesBaseline` and skip gracefully on ARM64 (where callees are inlined).

## When tests fail

### "No baseline found"

This happens when running on a new platform or runtime version. The test writes an `.actual`
file with the current JIT output.

**What to do:**
1. The structural validation (layer 2) has already passed at this point — the typeof
   elimination is confirmed. You are reviewing the ASM to establish an initial golden file.
2. Check the failure message — if it says "Output MATCHES existing baseline: ...", the JIT
   output is identical to another runtime version. Create a `.ref` file pointing to the
   existing `.asm` instead of duplicating it (the message includes the exact command).
3. If no match was found, verify the `.actual` file looks reasonable:
   - `visit-different-type`: should be tiny (just prologue+return, ~16 bytes ARM64) — the
     entire body was dead-branch eliminated
   - `visit-same-type`: should inline directly to Decrement with no typeof check
   - `enumerate-decrement-same-type`: full enumerator→visitor→Decrement pipeline inlined
   - `enumerate-decrement-mixed-type`: only same-type child path present, cross-type
     completely eliminated
   - `enumerate-decrement-multi-type`: two independent Decrement blocks (one per type),
     each loading a different table pointer
4. Rename `.actual` → `.asm` to establish the baseline (or create a `.ref` per step 2)
5. Commit the new baseline directory

### "JIT output differs from N known variant(s)"

This happens when the JIT produces a new encoding that doesn't match any known variant.
Structural validation has already passed at this point (typeof elimination confirmed).

**What to do:**
1. Review the `.actual` output — the failure message includes the ASM and a suggested command
2. If it's a valid alternative encoding (e.g., different addressing mode), add it as a variant:
   - The failure message tells you the exact filename (e.g., `{method}.v2.asm`)
   - If the output matches a variant in another version directory, create a `.ref` instead
3. Do NOT replace the existing `.asm` — it remains valid for other CI runner configurations
4. Do NOT delete any existing variants

### "Runtime correctness check failed"

This means the visitor produced wrong refcount results — the generated code is actually broken,
not just differently shaped. This is a real bug — investigate the visitor/enumerator code.

### "Expected N refcount decrement(s) but found M"

The structural validation detected a typeof elimination failure. The JIT either:
- Failed to eliminate a dead branch (more decrements than expected), or
- Failed to inline a Decrement call (fewer decrements than expected), or
- Emitted a runtime typeof comparison instead of constant-folding it

Investigate the visitor/enumerator code and check whether `[MethodImpl(AggressiveInlining)]`
is present on all Visit/Decrement methods. Also check that the types involved are structs
(the JIT can only constant-fold typeof for value types in generic contexts).

## CRITICAL: never delete old baselines

Old version directories (e.g., `osx-arm64-net10.0.2/`) remain valid for anyone running that
.NET version. When a new version appears, ADD a new directory alongside the old ones.

Only delete a baseline directory if that .NET version is no longer supported by the project.

## Adding new baseline scenarios

1. Add a new `[MethodImpl(MethodImplOptions.NoInlining)]` wrapper in
   `tests/Fabrica.JitBaseline/Program.cs` that exercises the method
2. Add runtime correctness assertions (AssertRefCount) before/after the call
3. Add a new `[InlineData]` entry in `JitBaselineTests.JitOutput_MatchesBaseline` with:
   - Method filter name matching the wrapper
   - Baseline file name (kebab-case)
   - Expected refcount decrement count (how many `RefCountTable.Decrement` calls should
     be fully inlined — determined by how many typeof branches survive constant-folding)
4. Run the tests — structural validation runs first (confirming typeof elimination), then
   baseline comparison fails and generates `.actual` files
5. Review and promote to `.asm` baselines

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CyrusNajmabadi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
