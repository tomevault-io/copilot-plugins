## bit

> How to run and monitor CLI tests for the bit project


# Testing Workflow

## Fixing a Bug

When fixing a bug, use this targeted approach:

1. **Build and install:**
   ```
   cabal install --overwrite-policy=always
   ```

2. **Run only the related test(s) first:**
   ```
   shelltest test/cli/specific.test --debug
   ```
   This gives fast feedback on your fix.

3. **Once the related tests pass**, run the full test suite (see below).

## Running Full Test Suite

When running the full test suite (new features, refactors, or final verification):

1. **Build and install first:**
   ```
   cabal install --overwrite-policy=always
   ```

2. **Delete stale output** (separate step — MUST succeed before running tests):
   ```
   Remove-Item test-output.txt -ErrorAction SilentlyContinue
   ```
   This prevents reading stale results from a previous run.

3. **Run parallel tests with PowerShell** (background immediately with `block_until_ms: 0`):
   ```
   powershell -ExecutionPolicy Bypass -File test\cli\run-parallel.ps1
   ```

4. Fix all failures, and go to step 3.

5. **Only after tests finish AND all fixes are complete**, do ONE final rebuild and full test run to verify everything.

## After Tests Pass

Once all tests pass, suggest a commit message:

- **One-line message** for most changes: `Fix/Add/Update <what changed>`
- **Multi-line message** only for major refactors or features that need explanation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/david-hoze) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
