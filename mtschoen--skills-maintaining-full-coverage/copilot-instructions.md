## skills-maintaining-full-coverage

> Use when: user mentions coverage, lint, linter, static analysis, code quality checks, 100% coverage, coverage gate, lint baseline, test report, TEST-REPORT.md, or 'is the build clean'; BEFORE declaring work done, summarizing what you built, or saying 'all passing/working/done/clean'; BEFORE committing or pushing; completing any feature/bugfix/refactor in a project that tracks test coverage OR has linters/analyzers configured (ruff, eslint, mypy, clang-tidy, jbinspect/inspectcode, golangci-lint, Roslyn analyzers, etc.); establishing coverage or lint tracking for a new project. If you wrote or changed production code, this skill applies - no exceptions (the Three Modes inside calibrate what the bar means for projects with pre-existing debt).


# Maintaining Full Coverage

## Overview

If the coverage report doesn't say 100% - or the linter has findings - you're not done. *(Unless the project never reached that bar and your current task isn't to get it there. Which of those you're in changes what "done" means - see Three Modes below.)*

**Core principle:** Every line of production code must be (a) exercised by a test and (b) clean against every linter/analyzer the project has configured. Uncovered lines are either untested (write a test) or unreachable (delete them). Lint findings are either real (fix them, ideally by restructuring) or genuine false positives (suppress per-case with explicit approval).

**Tests are not the only validators.** "Coverage" here means *covering the code with every check the project has* - tests for behavior, linters and analyzers for structure, type checkers for types. They're the same shape in the completion gate: machine-verifiable checked properties that must report clean before you declare done. Run them all, gate on all of them, restructure rather than suppress.

**Why this matters most:** Tests and linters earn their keep when they *accidentally surface real bugs*. A "unused variable" warning can reveal a typo that broke a code path. A `nullable` analyzer flagging a deref can pinpoint a real crash you missed. An uncovered branch can mean the condition is unreachable - i.e. dead code, often a bug. The discipline isn't paperwork; it's a structured way to make latent bugs visible. Treat findings as evidence first, noise second.

**This means ALL code, in ALL languages, in the ENTIRE repo.** A C# project with a C++ native library needs 100% coverage in both C# and C++, and Roslyn analyzers on the C# AND clang-tidy on the C++. A Python backend with a JavaScript frontend needs coverage.py + istanbul/c8 AND ruff + eslint. If production code exists in the repo and it's in a language you can compile/run, it needs coverage tooling, lint tooling, and tests. No language gets a pass.

**Violating the letter of this rule is violating the spirit of this rule.**

This skill is the final layer in a three-skill stack:

1. `test-driven-development` - writes tests before code
2. `verification-before-completion` - proves tests pass with evidence
3. `maintaining-full-coverage` - proves every line is covered AND every analyzer reports clean, and the report is updated

TDD is upstream discipline. Verification is evidence. This skill is the metric gate. When several completion skills are in play, the order is: this skill's gate (tests / lint / coverage) -> smoke-test -> docs-update -> declare done / commit.

## Three Modes: what "the bar" means here

The 100%-coverage / 0-findings bar is the destination. Whether *this task* must arrive there depends on where the project starts and what you were asked to do. **Read the project's current state first** - the checked-in `TEST-REPORT.md` (if any) plus a fresh coverage + lint run - then place yourself in one of three modes:

**1. Maintain - the project is already clean.** Coverage is 100% and every configured linter reports 0 findings (the report says so and a fresh run confirms it). The bar is absolute: hold it. Your change must not drop coverage below 100% and must not add a single finding. This is the strict gate - a regression here is blocked, full stop. Everything below about the completion gate and escalation ladder applies at full strength.

**2. Close the gap - reaching the bar IS the task.** The project isn't at 100% / 0 findings, and the user asked you to get it there ("add coverage", "clean up the lint", "get this to green", "set up the gate"). Then 100% / 0 is the deliverable and the full escalation ladder applies to every uncovered line and every finding - same strict bar as Maintain, because reaching it is the point.

**3. Best effort - the project is dirty and you're doing something else.** The project is below 100% / has findings, and your task is a feature, bugfix, or refactor - not a coverage/lint cleanup. Demanding the *whole repo* reach 100% before you can finish an unrelated feature is the hardline mistake this mode exists to prevent. The bar here is a **ratchet, not an absolute**:

- **Cover and clean what you touch.** New and changed production code gets tests and is lint-clean. You don't get to add debt just because debt already exists.
- **Don't regress the baseline.** Coverage percentage must not fall and the finding count must not rise versus the checked-in report. Those numbers are the floor.
- **Surface pre-existing debt; don't silently inherit it.** The 1,244 findings you didn't create aren't a blocker for *this* task - but record them in the report as the baseline and flag them (suggest a cleanup task). Don't pretend they're absent, and don't let them quietly grow.
- The escalation ladder still governs *your own* uncovered lines and findings. Best effort is not a license to skip testing the code you just wrote.

**When the mode is ambiguous, ask.** If the project is dirty and you can't tell whether the user wants gap-closing (mode 2) or best-effort-on-a-feature (mode 3), ask which - one short question. Don't silently default to the hardline (you'll block an unrelated task on inherited debt) and don't silently default to lax (you'll let a project that wanted cleanup stay dirty).

The report file records which mode applied and the baseline it measured against, so the next session knows whether a number is a ceiling to hold or a floor to ratchet up from.

## When to Use

**Before you declare anything "done":**

- You wrote or changed production code -> this skill applies
- You're about to say "all passing", "here's what changed", or summarize your work -> STOP, run the gate first
- You're about to commit or push -> STOP, run the gate first
- User asks you to commit -> this is a completion event, run the gate before the commit

**Always:** completing a feature, bugfix, or refactor; setting up coverage tracking for a new project; reviewing whether work is ready to commit.

**The test you wrote passing is not the finish line. 100% suite-wide coverage is the finish line** - in a project at or chasing that bar. In a below-bar project where coverage cleanup isn't the task, the finish line is: your new code covered, the baseline not regressed (see Three Modes).

**Throughout development (the nudge):**
While coding, periodically ask yourself: "If I ran coverage right now, would the code I just wrote be covered?" Every `if` has at least two paths. Every `try` has an `except`. Every early `return` has a condition that triggers it. Are both branches tested?

Don't batch all test-writing to the end. Write tests alongside code. Coverage debt compounds.

## The Completion Gate

One gate, two kinds of checker - coverage tools and linters/analyzers - run at the same moment (before declaring done, before committing, before saying "all passing/clean"), with the same steps:

```text
BEFORE claiming completion:

1. FIND the repo's coverage command AND its linters/analyzers. Check in order:
   a. AGENTS.md or CLAUDE.md - documented coverage/lint commands
   b. Scripts directory - run-coverage, coverage, lint, or test scripts
   c. Project config - pyproject.toml [tool.ruff]/[tool.mypy], package.json
      lint scripts, .clang-tidy, *.sln + .editorconfig (Roslyn),
      .resharper.dotsettings, golangci.yml, .rubocop.yml
   d. Pre-commit / CI config - .pre-commit-config.yaml, .github/workflows,
      .gitea/workflows, Makefile
   e. If no coverage command exists, construct one that covers ALL test
      projects in the repo (not just one). If the language has a standard
      linter (ruff, eslint, golangci-lint, clang-tidy, ...) and none is
      configured, ASK the human whether to add one - don't silently
      assume "this project doesn't lint."
2. VERIFY the commands cover all production code IN EVERY LANGUAGE.
   Do NOT trust AGENTS.md/CLAUDE.md or existing scripts blindly - they
   may be incomplete. Scan the repo for ALL production code; each
   language needs its own coverage tool AND its own linter.
   Example: C# managed code + a C++ native library needs BOTH
   dotnet-coverage AND gcov/llvm-cov, Roslyn analyzers AND clang-tidy.
   Example: a Python API + React frontend needs BOTH coverage.py AND
   istanbul/c8, ruff AND eslint. The frontend JS is not optional.
3. RUN everything - fresh, full repo, all configured rules, no cache.
   Slow analyzers (jbinspect/inspectcode, full-repo clang-tidy) can take
   minutes; run them anyway. "It's slow" is not in the Escalation Ladder.
4. READ the output - actual coverage percentage and uncovered lines;
   actual findings count, severity, locations (count findings, don't
   just check the exit code).
5. Is it 100% coverage AND 0 findings (read through the active mode -
   see Three Modes)?
   - YES -> continue to step 6
   - NO  -> enter the Escalation Ladder below.
           Do NOT claim completion. Do NOT skip to exclusions or
           suppressions.
6. WRITE or UPDATE `TEST-REPORT.md` at the repo root (format below),
   including the Lint section when any linter is configured. The report
   is a REQUIRED artifact - current git hash, test count, coverage
   numbers, per-tool findings.
7. COMMIT `TEST-REPORT.md` alongside your other changes.
8. ONLY after the report is written and committed: done.
```

Step 5 is written for Maintain and Close-the-Gap mode. In Best-Effort mode the test is different: *did I cover and clean the code I touched, and did I hold the baseline?* If yes, you're done for this task even though the absolute numbers are off the bar - record the baseline in the report (mode `best-effort`) and surface the remaining gap. Pre-existing findings in Maintain / Close-the-Gap count in full: 1000 inherited ruff warnings enter the Escalation Ladder the same way uncovered branches do.

## The Escalation Ladder

When the gate fails (coverage <100% or any lint finding), follow this order. **Never skip steps.**

**Step 1 - Write tests / fix findings.** Most uncovered lines are straightforwardly testable; most findings are straightforwardly fixable. Just do it.

**Step 2 - Heroic testing / restructuring.** Mock OS calls, simulate errors, use framework features creatively (see Heroic Coverage Scenarios). For findings, restructure the code so the analyzer's premise no longer holds (see Restructure Over Exclude). 100% / 0 is almost always achievable.

**Step 3 - Ask the human.** If you genuinely cannot cover a line or clear a finding, ask. Do not guess. Likely outcomes: the code is unreachable/dead -> **delete it** (dead code is a bug, not an exception); or the human knows a testing trick -> apply it.

**Step 4 - Framework exclusions / per-case suppressions.** `# pragma: no cover`, `/* istanbul ignore */`, `# noqa`, `[SuppressMessage]`, `NOLINT`. **Only with explicit human approval.** These produce a synthetic clean report. Never apply them silently, never mass-suppress a category.

**Step 5 - Documented exceptions.** Absolute last resort. The report file explicitly lists what's uncovered or unfixed and why. This becomes the new baseline that other work must meet.

After whichever step resolves it: update the report file, then done.

## Restructure Over Exclude

Before reaching for an exclusion (step 4) or a suppression, ask whether restructuring the production code eliminates the problem outright. When code tangles things that shouldn't be tangled - startup side effects, untestable singletons, network calls inline with business logic - refactor rather than excluding tests or weakening assertions. This is the same lever as deleting dead code: it improves the codebase, and restructuring to be testable is not overkill.

### Uncovered branches

A coverage tool counts a synthetic branch the cooperative-cancellation idiom never reaches. Restructure to remove the branch rather than excluding the line:

```kotlin
// Before: Kover counts the while-false branch as uncovered
while (isActive) { delay(1000); evictExpired() }

// After: no unreachable branch (delay() throws CancellationException on cancellation)
while (true) { delay(1000); evictExpired() }
```

The `while (isActive)` form has a false branch the tool can't reach (cancellation is by exception, never by loop exit); `while (true)` eliminates it without changing behavior. The same shape recurs across languages - Python's `asyncio.CancelledError` (`while running:` -> `while True:`) and .NET's `CancellationToken.ThrowIfCancellationRequested()` inside `while (true)` (replacing `while (!token.IsCancellationRequested)`).

### Analyzer suppressions

A warning you're tempted to silence often points at a real structural problem; restructuring to satisfy it can surface a genuine bug instead of hiding one. Never mass-suppress a category as "technically false positives" - demand a per-case justification for the rare real one.

In git-wizard (C#), a JetBrains `AccessToDisposedClosure` inspection flagged 18 sites. Bulk suppression was tempting; the structural fix was better. On the dominant pattern, moving the `using` inside the lambda makes test lifetime explicit in code instead of asserted in a comment:

```csharp
// Before - analyzer flags the disposable captured by the lambda
using var volume = new Volume(...);
Assert.ThrowsException<X>(() => volume.DoSomething());

// After - `using` inside the lambda; lifetime is explicit, warning gone
Assert.ThrowsException<X>(() => { using var volume = new Volume(...); volume.DoSomething(); });
```

For a callback stored on a longer-lived object, capture the **value** you need, not the disposable:

```csharp
var uiThreadId = dispatcher.UiThreadId;     // capture the value up front, not the disposable
obj.Callback = () => ranOnUi = Environment.CurrentManagedThreadId == uiThreadId;
```

Both eliminate the warning because the analyzer's premise no longer holds - and read better than the original. Suppression hides the smell; restructuring removes it.

### The legitimate exclusion case

Exclusion is correct for genuinely-untestable framework bindings - Android `MediaCodec`, `NSDManager`, XR Compose composables that require a running platform. Exclude those *specifically*, never whole-class blanket exclusions on production logic. If an exclusion attribute lands on a class whose name doesn't end in `Binding`, `Adapter`, or a similar platform-glue suffix, that's a smell that wants restructuring first.

## Report File Convention

Every project maintains a checked-in coverage report at the **repo root** named `TEST-REPORT.md`. If a project already has a report file under a different name, use that. Otherwise, create `TEST-REPORT.md`.

### Creating or updating the report

After running the gate, **you must** write or overwrite `TEST-REPORT.md` with the results. This is not optional - it is a required artifact of every run.

1. Get the current git short hash: `git rev-parse --short HEAD`
2. Get the total test count from the test runner output
3. Get line/branch/method coverage from the coverage tool output
4. Write `TEST-REPORT.md` at the repo root using the format below
5. Stage and commit it alongside your other changes

### Minimal required format

```text
<project> test report - <ISO 8601 timestamp>
===========================================

Status:   PASS | FAIL
Mode:     maintain | close-the-gap | best-effort
Tests:    <total> total
Git:      <short hash> (<branch or commit message>)
Coverage: <covered>/<total> statements (<pct>%)
          <N> lines uncovered
          <N> exclusion annotations
Lint:     <tool>: <N> findings (<N> errors, <N> warnings)
          [one line per configured tool]
          <N> per-case suppressions
          <N> documented exceptions
```

The **Lint** block is required when the project has any linter or analyzer configured. List every configured tool - `ruff`, `mypy`, `eslint`, `clang-tidy`, `inspectcode`/`jbinspect`, `golangci-lint`, Roslyn analyzers, etc.

The **Mode** line records which of the Three Modes governed this run, so a future reader knows whether a number is a ceiling to hold or a floor to ratchet up from. `Status` is defined relative to that mode:

- **maintain / close-the-gap:** `PASS` only when coverage is 100% AND every tool reports 0 findings (per-case suppressions and documented exceptions count as cleared, same as `pragma: no cover` does for coverage).
- **best-effort:** `PASS` when your changed code is covered and lint-clean AND the baseline did not regress (coverage % didn't drop, finding count didn't rise). Record the inherited baseline (e.g. `Coverage: 312/400 (78%) - baseline held`, `Lint: eslint 1244 findings (pre-existing baseline, +0 this change)`) so the ratchet is auditable. A best-effort `PASS` is honest about not being at the absolute bar; it is not a `FAIL`.

Beyond the minimum, projects add whatever is useful - per-suite breakdowns, branch coverage, UI audit stats, timing.

### Report file rules

- **Lives at repo root as `TEST-REPORT.md`** unless the project already has a report file elsewhere.
- **Checked into the repo.** Tracked in git history. `git diff` on the report instantly shows regressions.
- **Updated whenever tests or coverage change.** Not "later" - now, as part of the work.
- **Git hash above coverage results.** It establishes what code the numbers describe.
- **AGENTS.md (or CLAUDE.md) documents the coverage command** and references `TEST-REPORT.md`.
- **With CI:** PRs that regress coverage are rejected unless an exemption grants a new baseline.
- **Without CI:** Honor system, but git history still catches regressions.

## Heroic Coverage Scenarios

100% is almost always achievable. These patterns prove it.

- **OS/platform-specific code:** mock `platform.system()`, `Path.read_text()` with `PurePosixPath` comparison, `os.execv()`. Test both branches even on one OS.
- **Error paths requiring external failures:** mock the dependency - database errors, network timeouts, permission denied. The error handler exists because it can happen. Simulate it.
- **Elevated/admin-only code paths:** mock the privilege check to test both paths. For things that genuinely cannot be mocked (e.g., UAC prompts), interactive tests are an option: show an instructional dialog ahead of the system prompt ("you should say yes to this one") so the human knows what to do during the test run.
- **Browser/integration coverage:** Puppeteer/Playwright tests hitting every route and handler. UI audit scripts tracking which pages, functions, and handlers are exercised.
- **Startup/shutdown code:** test initialization with mocked dependencies; trigger cleanup/teardown paths explicitly.

If you think a line is untestable, you are probably wrong. Mock harder, simulate the condition, or ask the human - they may know a trick, or the code might be dead and should be deleted.

## Rationalization Table

| Excuse | Reality |
|--------|---------|
| "That line is unreachable" | Then delete it. Dead code is a bug, not an exception. |
| "It's just error handling / platform-specific code" | Error handlers exist because errors happen; platform checks have two branches. Mock the error, mock the platform, test both. |
| "Coverage is 98%, close enough" | 98% means uncovered lines. Find them. Test them. |
| "I'll add tests later" | Later never comes. The gate is now. |
| "This is just config/glue code" | Config can break. Glue can fail. Test it. |
| "The framework makes this untestable" | Ask the human. They may know a trick, or the code should be restructured. |
| "Adding `pragma: no cover` is faster / asking takes longer" | Exclusions require human approval. If you can fix it, fix it. If you can't, ask - don't reach for pragma instead of asking. |
| "I'll update the report file after" | The report is a first-class artifact. Update it now, commit it with your changes. |
| "Both branches do the same thing, testing one is enough" | The coverage tool disagrees. Test both. |
| "The C++/JS/other-language code is a separate concern" / "I got 100% on the main language" | 100% of ONE language is not 100%. Every language in the repo needs its own coverage and lint tooling. |
| "It's just a lint warning / a false positive" | Often the linter found something you missed - investigate before suppressing. Real false positives get restructured around or per-case suppressed with approval, never mass-suppressed. |
| "Lint debt is pre-existing, not my problem" | Depends on mode. Maintain / close-the-gap: it's your problem - enter the Escalation Ladder. Best-effort: don't add to it, don't let the count grow, record + surface the baseline. |
| "Running jbinspect / clang-tidy / mypy is too slow" | Run it anyway. "Slow" is not a step in the Escalation Ladder. If you must defer, raise it with the human - don't skip silently. |
| "The project doesn't lint" | Did you check? `pyproject.toml`, `package.json`, `.clang-tidy`, `*.sln`, `.pre-commit-config.yaml`, CI workflows - verify before assuming. |

## Red Flags - STOP and Reconsider

- Reaching for `pragma: no cover` / `# noqa` / `eslint-disable` / `[SuppressMessage]` / `NOLINT` before attempting to test or restructure, or before asking the human
- Claiming completion with uncovered lines or findings you haven't investigated
- Assuming code is unreachable without verifying (it might be dead - delete it)
- Batching all test-writing to the end
- Treating `TEST-REPORT.md` as optional, or claiming completion without updating it
- Skipping straight to step 4 or 5 of the escalation ladder
- Writing tests that cover the line but don't test meaningful behavior
- Forgetting to test both branches of a conditional
- Declaring 100% coverage or "lint clean" when you only checked one language in a multi-language repo
- Declaring "lint clean" without actually running the linter
- Skipping the lint gate because "the project doesn't lint" without verifying via project config / CI

---
> Source: [mtschoen/skills-maintaining-full-coverage](https://github.com/mtschoen/skills-maintaining-full-coverage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
