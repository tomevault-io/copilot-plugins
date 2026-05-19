## freelang

> This guide documents processes and patterns for AI agents/collaborators working on the Freelang compiler project.

# Agent Guide for Freelang Development

This guide documents processes and patterns for AI agents/collaborators working on the Freelang compiler project.

## Table of Contents

1. [Testing & Debugging](#testing--debugging)
2. [Windows PE Backend Development](#windows-pe-backend-development)
3. [Common Issues & Solutions](#common-issues--solutions)
4. [Project Structure](#project-structure)

## Testing & Debugging

### Running Tests

The project has multiple ways to run tests:

1. **Full test suite** (expensive, ~2-5 minutes):
   ```powershell
   pwsh ./tests/run-all.ps1 > test-output.txt 2>&1
   ```
   Always save output to a file to enable analysis without re-running.

2. **Subset of tests**:
   ```powershell
   pwsh ./tests/run-all.ps1 --hide-passes
   ```

3. **Single test file**:
   ```powershell
   pwsh flx.ps1 tests/test-name.flx
   ```

### Analyzing Test Failures

After running the full suite with output saved:

1. **Check summary** (last 10-20 lines):
   ```powershell
   Get-Content test-output.txt -Tail 20
   ```

2. **Count specific error patterns**:
   ```powershell
   # Count access violations (crashes)
   Get-Content test-output.txt | Select-String "Actual:\s+-1073741819" | Measure-Object

   # Find linker errors
   Get-Content test-output.txt | Select-String "unresolved external symbol"
   ```

3. **Common exit codes on Windows**:
   - `-1073741819` (0xC0000005): Access violation / segfault
   - `1120`: Linker error (unresolved symbols)
   - `0`: Success

### Debugging Individual Tests

Start with the simplest failing test to understand the root cause:

```powershell
# Test a basic case
pwsh flx.ps1 tests/assert_pass.flx

# Test specific features
pwsh flx.ps1 tests/arrays-basic.flx
pwsh flx.ps1 tests/ir-string-concat.flx
```

Look for linker errors in the output - these directly indicate missing implementations.

## Windows PE Backend Development

### Architecture Overview

The Windows PE backend is in `backend/codegen-x86_64-pe.js` and generates x86-64 assembly for Windows using:
- **Calling Convention**: Microsoft x64 ABI (rcx, rdx, r8, r9 for args)
- **Linker**: lld-link (LLVM linker)
- **Target**: clang -target x86_64-pc-windows-msvc
- **Dependencies**: Only Kernel32.dll (no libc)

### Key Differences from macOS Backend

| Aspect | macOS (System V ABI) | Windows (MS x64 ABI) |
|--------|---------------------|---------------------|
| Args 1-2 | rdi, rsi | rcx, rdx |
| Args 3-4 | rdx, rcx | r8, r9 |
| Stack args | 8(%rsp)+ | 32(%rsp)+ (shadow space) |
| Shadow space | None | 32 bytes required |

### Adding Missing Intrinsics

When you see linker errors like:
```
unresolved external symbol _rt_string_concat
unresolved external symbol _rt_array_concat
unresolved external symbol _rt_object_merge
unresolved external symbol _rt_value_shape_plus
```

**Step 1: Check if the `_freelang_*` version exists**

The codebase uses both naming conventions:
- `_rt_*`: Called from IR's `CallRuntime` instructions
- `_freelang_*`: Internal implementation names

Search for existing implementations:
```javascript
// In backend/codegen-x86_64-pe.js
function emitStringConcat(asm) {
  asm.label('_freelang_string_concat');
  // Implementation...
}
```

**Step 2: Add runtime aliases**

Create label aliases that jump to the implementation:
```javascript
function emitRuntimeAliases(asm) {
  asm.label('_rt_string_concat');
  asm.instr('jmp', '_freelang_string_concat');

  asm.label('_rt_array_concat');
  asm.instr('jmp', '_freelang_array_concat');

  // ... etc
}
```

**Step 3: Implement missing `_freelang_*` functions**

If the `_freelang_*` implementation is missing or stubbed:

1. Check the macOS backend (`backend/codegen-x86_64-macho.js`) for reference
2. Adapt the calling convention from System V to MS x64
3. Key changes needed:
   - Replace `%rdi, %rsi` with `%rcx, %rdx` for first 2 args
   - Add shadow space: `subq $40, %rsp` at function start
   - Use `callq _freelang_memcpy` instead of `rep movsq` if needed

Example adaptation (array_concat):
```javascript
// macOS version uses:
asm.instr('movq', '%rdi', '%r8');    // arr1 from rdi
asm.instr('movq', '%rsi', '%r9');    // arr2 from rsi

// Windows version uses:
asm.instr('movq', '%rcx', '%r8');    // arr1 from rcx
asm.instr('movq', '%rdx', '%r9');    // arr2 from rdx
```

**Step 4: Update buildFromIR to emit functions**

In the `buildFromIR` function, ensure the new intrinsics are emitted:

```javascript
// Around line 266-286
const needsValueOps = features.arrays || features.strings || features.objects;
if (needsValueOps) {
  emitArrayConcat(asm);
  emitStringConcat(asm);
  emitObjectMerge(asm);
  emitRuntimeAliases(asm);
  emitValueShapePlus(asm);
}
```

**Important**: `_rt_value_shape_plus` calls all three concat/merge functions, so they must all be emitted together even if only one feature is used.

### Implementing _rt_value_shape_plus

This intrinsic implements dynamic `+` operator dispatching:

```javascript
function emitValueShapePlus(asm) {
  asm.label('_rt_value_shape_plus');

  // 1. Check if both operands are arrays -> call _freelang_array_concat
  // 2. Check if both operands are strings -> call _freelang_string_concat
  // 3. Check if both operands are objects -> call _freelang_object_merge
  // 4. Fallback: return left operand

  // Type tags:
  // 1 = array, 2 = object, 3 = string
  // Check type_tag at offset 0 in heap object
}
```

### Memory Layout

Heap objects (arrays, strings, objects) have this structure:
```
Offset  | Content
--------|------------------
0       | Type tag (1=array, 2=object, 3=string)
8       | Size in words (including header)
16      | Length (element count for arrays/strings)
24+     | Data (elements/characters/fields)
```

All heap pointers are tagged (LSB = 1). Untag with `andq $-2, %rax`.

## Common Issues & Solutions

### Issue: Tests crash with exit code -1073741819

**Cause**: Access violation - likely dereferencing null/invalid pointer or stack corruption

**Debug**:
1. Check if calling convention is correct (shadow space, register usage)
2. Verify stack alignment (must be 16-byte aligned before `callq`)
3. Check that all callee-saved registers (rbx, r12-r15, rbp) are preserved

### Issue: Linker error "unresolved external symbol"

**Cause**: Missing intrinsic implementation or missing runtime alias

**Fix**: See "Adding Missing Intrinsics" section above

### Issue: Test produces wrong output but doesn't crash

**Cause**: Logic error in intrinsic implementation

**Debug**:
1. Compare assembly output with macOS backend
2. Check if type tags are handled correctly
3. Verify untagging/retagging of pointers
4. Ensure lengths are calculated correctly

### Issue: Stack corruption / random crashes

**Cause**: Shadow space not allocated, or stack not 16-byte aligned

**Fix**:
```javascript
// Always allocate shadow space (minimum 32 bytes)
// Round up to 16-byte alignment: use 40, 56, 72, etc.
asm.instr('subq', '$40', '%rsp');

// Before any callq, ensure proper alignment
// After function entry, stack should be misaligned by 8
// (because call pushes return address)
// So allocate: (needed_space + 8) rounded to 16
```

## Project Structure

```
freelang/
├── backend/
│   ├── codegen-x86_64-pe.js      # Windows PE backend
│   ├── codegen-x86_64-macho.js   # macOS backend (reference)
│   └── codegen-x86_64-minimal.js # Minimal backend
├── compiler-frontend.js           # Parsing & semantic analysis
├── compiler-ir.js                 # IR generation
├── tests/                         # Test files (.flx)
│   ├── run-all.ps1               # Windows test runner
│   └── run-all.sh                # Unix test runner
├── flx.ps1                        # Windows compile & run script
└── flx.sh                         # Unix compile & run script
```

### Key Files to Understand

1. **backend/codegen-x86_64-pe.js**: Core code generation
   - `buildFromIR()`: Main entry point
   - `emitInstr()`: Instruction emission
   - `emit*()` functions: Intrinsic implementations

2. **compiler-ir.js**: IR structure
   - Defines IR instruction types (CallIntrinsic, CallRuntime, etc.)
   - Useful for understanding what intrinsics are needed

3. **tests/**: Look at simple tests first
   - assert_pass.flx: Minimal working test
   - arrays-basic.flx: Array operations
   - ir-string-concat.flx: String concatenation

## Workflow for Fixing Test Failures

1. **Run full test suite, save output**
   ```powershell
   pwsh ./tests/run-all.ps1 > test-output.txt 2>&1
   ```

2. **Analyze failure patterns**
   - Group by error type (linker errors vs crashes vs wrong output)
   - Identify lowest-hanging fruit (missing intrinsics are easiest)

3. **Fix linker errors first**
   - These indicate missing implementations
   - Usually fix 20-30 tests at once
   - Example: implementing _rt_value_shape_plus fixed all + operator tests

4. **Test individual fixes**
   ```powershell
   pwsh flx.ps1 tests/specific-test.flx
   ```

5. **Re-run full suite to measure progress**
   - Compare pass/fail counts
   - Identify new failure patterns

6. **Fix crashes next**
   - Debug calling convention issues
   - Check stack alignment and shadow space
   - Verify register preservation

7. **Fix wrong output last**
   - Logic errors in intrinsics
   - Type handling bugs
   - Edge cases

## Recent Fixes (2025-12-07)

### Implemented Missing Runtime Intrinsics

**Problem**: 145 tests failing, many with linker errors for:
- `_rt_value_shape_plus`
- `_rt_array_concat`
- `_rt_string_concat`
- `_rt_object_merge`

**Solution**:
1. Implemented `emitArrayConcat()` - was just a stub returning immediately
2. Created `emitRuntimeAliases()` to provide `_rt_*` labels
3. Implemented `emitValueShapePlus()` for dynamic `+` operator
4. Updated `buildFromIR()` to emit these functions together

**Files Changed**: `backend/codegen-x86_64-pe.js`

**Impact**: Fixed linker errors for ~30+ tests involving string/array/object concatenation

**Key Insight**: The `_rt_value_shape_plus` function needs all three concat/merge functions available because it dispatches to them based on runtime type checking. They must be emitted as a group.

---

## Tips for AI Agents

1. **Always save test output** - full runs are expensive, don't waste them
2. **Start with linker errors** - they directly tell you what's missing
3. **Use macOS backend as reference** - it's more complete
4. **Test incrementally** - verify each fix before moving to the next
5. **Document as you go** - update this file with new patterns
6. **Use the docs index** - `docs/INDEX.md` has the current map
7. **Stdlib docs entry** - `stdlib/docs/README.md` is the map for stdlib surfaces
7. **Check calling convention** - most bugs come from wrong registers
8. **Verify stack alignment** - Windows requires shadow space + 16-byte alignment
9. **Chaos world model** - treat chaos tags as modeled world states (data), not bugs; reserve `fall` for invariants

## Useful Commands Reference

```powershell
# Run all tests, save output
pwsh ./tests/run-all.ps1 > test-output.txt 2>&1

# Get test summary
Get-Content test-output.txt -Tail 20

# Test single file
pwsh flx.ps1 tests/test-name.flx

# Find all .flx files
ls tests/*.flx

# Search for pattern in backend
Select-String "pattern" backend/codegen-x86_64-pe.js

# Count specific errors
Get-Content test-output.txt | Select-String "pattern" | Measure-Object
```

## Latest Test Snapshot
- 2026-01-02: macOS full suite `./tests/run-all.sh` → **312/312 passing** (272 normal, 40 expected-fail). Output saved as `test-output.txt`.

## Agent Log (Windows PE)

### 2025-12-20

- **str_len on ints**: Windows PE backend did not mirror Mach-O/minimal behavior. `str_len` should accept either a string or a boxed int; if int, it must call `_freelang_int_to_string` and then return the string length. Fix implemented in `backend/codegen-x86_64-pe.js` in the `CallIntrinsic` handling (special-case `str_len`).
- **Fork crash hypothesis**: `simple_weave` and `test-fork-*` still crash with access violation (`-1073741819`). The crash happens after initial output, so likely inside `_rt_fork_job` path.
  - `GetModuleFileNameA` was previously called with wrong register order; fixed to use RCX (hModule = 0), RDX (buffer), R8 (size).
  - Added RSI/RDI preservation in `_rt_fork_job` (callee-saved on Windows x64). Stack alignment should remain intact (8 extra pushes = 64 bytes).
  - If crashes persist, next suspects: incorrect command line buffer layout in `_rt_fork_job` (rsp+256 base), missing zeroing of PROCESS_INFORMATION for this path, or shadow-space/stack arg overlap in CreateProcessA call.
- **Test runner insight**: The JS compare tool normalizes CRLF to LF on Windows, so output diffs are not line-ending driven. Failures indicate missing output or early crash, not CRLF mismatch.

### 2025-12-20 (later)

- **CreateProcess crash root cause**: `_freelang_create_job_dirs` saved `%rsi`/`%rdi` but did not restore them. The missing `popq %rdi` / `popq %rsi` corrupted the stack and crashed on return during fork setup. Restoring both fixed `simple_weave` and `test-fork-*` crashes on Windows.
- **Debugging caution**: Inline debug markers that emit `.ascii` need a jump over the embedded data, and any debug call that uses `WriteFile` will clobber `%rax` (and other volatile regs). Save/restore or move debug markers after consuming critical return values.

### 2025-12-26

- **Non-deterministic print order**: On Windows, print statements may appear in non-deterministic order due to buffering or process timing. If tests fail with "stdout mismatch" but correct content, create an `.expect.out_unordered` file instead of fixing code. Example: `if_else_scopes.flx` expects "100, 10, 999" but may get "10, 100, 999" - both are valid.

- **Register clobbering in `_freelang_input`**: When `_freelang_input` creates a string, it calls `_freelang_alloc_words` which clobbers volatile registers (R9, R10, R11 per Windows x64 ABI). Must save R9 (length) and R10 (buffer ptr) to stack before alloc call, then restore after. Fixed by adding:
  ```javascript
  asm.instr('movq', '%r9', '304(%rsp)');   // save length
  asm.instr('movq', '%r10', '312(%rsp)');  // save buffer ptr
  asm.instr('callq', '_freelang_alloc_words');
  asm.instr('movq', '304(%rsp)', '%r9');   // restore
  asm.instr('movq', '312(%rsp)', '%r10');  // restore
  ```

- **ReadFile DWORD issue**: `ReadFile` writes bytes_read as DWORD (4 bytes), not QWORD. Use `movl` to read it, not `movq`, or upper 32 bits contain garbage.

- **Test efficiency**: Run specific tests with `pwsh ./run-all.ps1 test1.flx test2.flx` instead of full suite. Always redirect full suite output to a file for later analysis.

## Implementation Guidance

Before making changes, read `docs/dev/case-study-bytes-slot-dispatch.md` for a worked example of how to evaluate implementation choices against freelang philosophy. Key principles:

1. **Extend existing infrastructure** - don't create parallel mechanisms
2. **Prefer minimal blast radius** - backend-only beats IR+backend
3. **Avoid hidden costs** - don't add runtime overhead to unrelated paths
4. **Keep explicit paths** - specialized types can have explicit accessors

## Current Notes
- Bytes slot access (`data\\i`) now works via dynamic dispatch; see `docs/dev/case-study-bytes-slot-dispatch.md`

---
> Source: [DO-SAY-GO/freelang](https://github.com/DO-SAY-GO/freelang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
