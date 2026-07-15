## testing-requirements

> **ABSOLUTE RULE: BUILD AND ALL TESTS MUST PASS BEFORE ANY COMMIT OR PR**

# Testing Requirements

## CRITICAL: Build & Test Success Required

**ABSOLUTE RULE: BUILD AND ALL TESTS MUST PASS BEFORE ANY COMMIT OR PR**

### Pre-Commit Checklist (MANDATORY)
Before ANY commit or PR creation, you MUST:

1. ✅ **Run `npm run build`** - Build MUST succeed with exit code 0
   - No TypeScript errors allowed
   - No webpack errors allowed
   - If build fails, fix ALL errors before proceeding

2. ✅ **Run `npm test`** - ALL tests MUST pass (100% pass rate)
   - Check the test summary at the end of output
   - Verify "Tests: X passed, X total" (no failures or errors)
   - If ANY test fails, fix it before proceeding

3. ✅ **Verify exit codes**
   - `npm run build` exit code must be 0
   - `npm test` exit code must be 0

### NEVER:
- ❌ Commit code that doesn't build
- ❌ Commit code with failing tests
- ❌ Push PRs without running build first
- ❌ Push PRs without running tests first
- ❌ Assume tests pass without actually running them
- ❌ Congratulate yourself on "some passing tests" when others fail

### Required Workflow
```bash
# MANDATORY before every commit:
npm run build  # Must succeed with exit code 0
npm test       # Must show 100% pass rate with exit code 0

# Only then:
git add -A
git commit -m "message"
git push
```

## Mandatory Test Standards

### ALL TESTS MUST PASS BEFORE COMMIT
- **100% test pass rate required** - No commits or PRs with failing tests
- **No exceptions** - If tests fail, they must be fixed before committing
- **No partial credit** - "58/73 tests passing" means 15 tests FAILING, which is unacceptable

### Before Every Commit
1. **Run build**: `npm run build`
2. **Run all tests**: `npm test`
3. **Verify 100% pass rate** - Check output shows "Tests: X passed, X total" with no failures
4. **Fix all failures** - Do not proceed until build and all tests pass
5. **Re-run after fixes** - Ensure fixes didn't break other tests

### Test Failure Response
When tests fail:
1. **Stop immediately** - Do not commit, do not create PR
2. **Analyze failures** - Read error messages carefully
3. **Fix the issues** - Update code or tests as needed
4. **Verify fixes** - Run tests again until 100% pass
5. **Only then proceed** - Commit only after all tests pass

### What Counts as a Test Failure
- ❌ Test fails with error or assertion failure
- ❌ Test times out
- ❌ Test is skipped due to errors (not intentional .skip())
- ❌ Test infrastructure errors (database setup, mocks, etc.)
- ✅ Only passing tests count as success

### Acceptable Test States for Commit
- ✅ All tests pass (green)
- ✅ Tests intentionally skipped with .skip() or .todo() (document why)
- ❌ NEVER commit with failing tests
- ❌ NEVER commit with test errors
- ❌ NEVER commit with "most tests passing"

### Test Quality Standards
- Tests must be deterministic (same result every run)
- Tests must clean up after themselves (no side effects)
- Tests must not depend on external services in CI
- Tests must run in reasonable time (< 1 minute for unit tests)

### Documentation of Test Status
When creating commits/PRs:
- ✅ "Build successful, all tests passing (73/73)" - Acceptable
- ✅ "Tests: 73 passed, 5 skipped (documented), 78 total" - Acceptable
- ❌ "Tests: 58/73 passing" - NOT ACCEPTABLE
- ❌ "Most tests passing, some infrastructure issues" - NOT ACCEPTABLE
- ❌ "Build fails but tests pass" - NOT ACCEPTABLE

## Consequences of Violating This Rule
- Commits with failing builds or tests must be fixed immediately
- PRs with failing builds or tests will be rejected
- Code with failing tests is not production-ready regardless of implementation quality

## Summary
**Zero tolerance for failing builds or tests. 100% success rate required before any commit or PR.**

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
