## feature-dev

> /feature-dev — Autonomous feature development pipeline with TDD-first workflow. Takes a feature description in an existing project and drives it through Spec → Tests → Implementation → Review → PR without user interaction. Supports GitHub Copilot (VS Code) and Cursor.


# /feature-dev — Autonomous Feature Development Pipeline

## Overview

`/feature-dev <feature description>` launches a fully autonomous 6-phase pipeline that takes any feature request in an existing project from description to merged PR — **without requiring any user interaction during execution**.

Key differentiator: **TDD-first** — failing tests are written before a single line of implementation code. The pipeline never ships code that doesn't pass its own tests.

Designed to run as one of many parallel agents. The user can spin up best-of-N instances and pick the best PR.

## Activation

```
/feature-dev <feature description>
/feature-dev <feature description> --branch <branch-name>
/feature-dev <feature description> --tag <tagname>
/feature-dev <feature description> --base <base-branch>
```

Examples:
- `/feature-dev "add dark mode toggle to settings page"`
- `/feature-dev "rate limit API endpoint" --tag rate-limit-v1`
- `/feature-dev "export data as CSV" --branch feat/csv-export --base develop`

## ⚠️ Autonomous Operation Declaration

**This skill operates in fully autonomous mode by design.**

- ❌ **Never** call `ask_user`, `AskQuestion`, `AskUserQuestion`, or any interactive prompt during Phases 0–6
- ✅ When ambiguous: document 2–3 interpretations, choose the most aligned with existing codebase patterns
- ✅ When blocked by a missing dependency: document in spec, stub it, continue
- ✅ When tests fail after implementation: retry up to 3 times, log as `mistake` in memory if unresolved
- ✅ Every significant decision is logged in `FEATURE_LOG.md` for best-of-N run comparison

The user is expected to be away. When they return, they review the PR description and `FEATURE_LOG.md`.

---

## Memory Protocol

### Shared Memory with /incubate

`/feature-dev` shares the same `user-memory.jsonl` as `/incubate`. This means:
- User preferences learned during incubation apply to feature dev
- Feature development insights are visible to future incubation runs

### Storage Paths

| Platform | Memory Path |
|----------|-------------|
| GitHub Copilot (VS Code) | `~/.copilot/skills/incubate/data/user-memory.jsonl` |
| Cursor | `~/.cursor/skills/incubate/data/user-memory.jsonl` |
| Windows (Copilot) | `%USERPROFILE%\.copilot\skills\incubate\data\user-memory.jsonl` |
| Windows (Cursor) | `%USERPROFILE%\.cursor\skills\incubate\data\user-memory.jsonl` |

### Memory Rules

- Load at Phase 0; apply `preference` and `pattern` entries throughout execution
- Save at Phase 6 (minimum 3 new entries of type `feature`)
- Entry types added by this skill: `feature` (new type alongside existing types)
- Archive when JSONL exceeds 200 entries (move oldest 100 to `*-archive.jsonl`)

### Platform Detection

| Signal | Platform | Repo Operations |
|--------|----------|-----------------|
| `github-create_branch` MCP tool available | Copilot (VS Code) | GitHub MCP tools |
| Tool unavailable + `git` in PATH | Cursor / Claude Code | git CLI |
| Neither available | Unknown | Local folder only |

---

## Phase 0: Setup

**Goal:** Parse feature request, load memory, detect platform, explore codebase, create feature branch.

### Steps

1. **Parse Invocation**
   - Extract feature description from invocation
   - Generate feature slug: lowercase, replace spaces with `-`, strip special chars, truncate to 40 chars
   - If `--branch` provided: use as branch name override
   - If `--tag` provided: use as slug override
   - If `--base` provided: use as base branch; otherwise detect default branch (`main`, `master`, or `develop`)
   - If slug already exists as a branch: append `-v2` (or `-v3` etc.) and continue

2. **Load Memory**
   - Check if `user-memory.jsonl` exists at the platform path
   - If exists: parse all `active` entries, build working context
   - If not exists: create directory structure, initialize empty file
   - If corrupted: rename to `<name>.corrupted.<timestamp>`, create fresh file

3. **Detect Platform**
   - Test if `github-create_branch` MCP tool is available → `PLATFORM=copilot`
   - Else test if `git --version` succeeds → `PLATFORM=cursor`
   - Else → `PLATFORM=local`
   - Log to `FEATURE_LOG.md`

4. **Explore Codebase**
   - List top-level directory structure
   - Identify: primary language, test framework, test directory location, code style
   - Apply memory `preference` entries (e.g., preferred language, test conventions)

5. **Create Feature Branch**
   - **Copilot path:** `github-create_branch(branch="feat/<slug>", from_branch=<base>)`
   - **Cursor/git path:** `git checkout -b feat/<slug> <base>`
   - Log branch name to `FEATURE_LOG.md`

6. **Initialize FEATURE_LOG.md**
   ```markdown
   # Feature Log: <slug>
   
   - Feature: <description>
   - Branch: feat/<slug>
   - Base: <base-branch>
   - Platform: <copilot|cursor|local>
   - Started: <ISO timestamp>
   - Phase: 0
   ```

**Exit Criteria:** Memory loaded, platform detected, feature branch created, FEATURE_LOG.md initialized.

---

## Phase 1: Feature Spec

**Goal:** Write a tight 5-section feature specification without user input.

### Feature Spec Template

```markdown
# Feature Spec: <Feature Name>

## 1. Problem Statement
One paragraph: what specific pain does this solve? Who experiences it? What's the current workaround?

## 2. Acceptance Criteria
Numbered list of testable conditions that define "done". Each criterion must be verifiable by an automated test or observable UI state.

Example:
1. Given X, when Y, then Z
2. The endpoint returns 4xx when ...
3. The UI displays ... when ...

## 3. Scope Boundary
**In scope:** (bullet list — max 5 items)
**Out of scope:** (bullet list — explicit exclusions to prevent creep)

## 4. Technical Approach
2–4 sentences: what changes (files/components/APIs), which patterns to follow, any non-obvious design decisions.

## 5. Autonomous Interpretation
(a) If the feature description was ambiguous, list 2–3 possible interpretations
(b) Chosen interpretation
(c) Rationale (aligns with codebase patterns, most commonly requested, etc.)
```

### Steps

1. Analyze feature description using loaded memory (apply relevant `preference` and `pattern` entries)
2. Explore codebase to understand existing patterns for Section 4
3. Fill each section — minimum 2 sentences per section
4. For Section 5: always populate even for clear descriptions (document assumptions)
5. Save to `FEATURE_SPEC.md` in current directory (or project root)
6. Append to `FEATURE_LOG.md`: `Phase 1: Spec written`

**Exit Criteria:** FEATURE_SPEC.md exists, all 5 sections present, each ≥ 2 sentences.

---

## Phase 2: TDD Design

**Goal:** Write failing tests that precisely map to Acceptance Criteria before any implementation code exists.

### TDD First Principle

> **Write tests first. All tests must be RED before Phase 3 begins.**

This is non-negotiable. Tests written after implementation are less likely to catch edge cases and create false confidence.

### Test Coverage Requirements

| Acceptance Criterion | Required Test Cases |
|---------------------|---------------------|
| Happy path | 1 test per criterion minimum |
| Edge cases | At least 2 edge cases total |
| Error cases | 1 test per documented error mode |
| Integration | 1 smoke test end-to-end |

### Framework Detection

Detect test framework from codebase (apply memory preferences):

| Signal | Framework |
|--------|-----------|
| `pytest`, `test_*.py`, `*_test.py` | pytest (Python) |
| `jest.config.*`, `*.test.{js,ts}` | Jest (JS/TS) |
| `cargo test`, `#[test]` | Rust test |
| `go test`, `*_test.go` | Go test |
| `junit`, `@Test` | JUnit (Java) |
| None detected | Write framework-agnostic tests, document assumption |

### Steps

1. Identify test directory from codebase (e.g., `tests/`, `__tests__/`, `spec/`)
2. Create test file: `tests/test_<slug>.{py|js|ts|rs|go}` (match existing convention)
3. Write one test function per Acceptance Criterion from Phase 1
4. Write at least 2 edge case tests and 1 error case test
5. Run tests to confirm they are **RED** (failing — this is correct at this stage):
   - **Copilot path:** Use `powershell` or `bash` tool to run test command
   - **Cursor path:** Use terminal / run tool
6. Save test run output showing failures to `FEATURE_LOG.md`
7. Append to `FEATURE_LOG.md`: `Phase 2: Tests written — N tests RED (expected)`

**Exit Criteria:** Test file exists with ≥ (N acceptance criteria + 3) tests. Tests are RED.

---

## Phase 3: Implementation

**Goal:** Write implementation code to make all tests GREEN.

### Self-Test Checklist

Before marking Phase 3 complete, verify:
- [ ] All tests from Phase 2 pass (GREEN)
- [ ] No hardcoded credentials, tokens, or secrets
- [ ] Basic error handling present for each error mode documented in spec
- [ ] Code follows existing style patterns (detected in Phase 0 codebase exploration)
- [ ] No dead code or debug prints in committed files
- [ ] Works without modification on target platform(s)

### RED → GREEN Protocol

```
1. Run tests (all RED) — confirm starting point
2. Implement minimal code to make first test pass
3. Run tests again
4. Repeat until all tests GREEN
5. If a test cannot be made to pass:
   - Diagnose root cause
   - If genuine scope issue: update spec and test to note limitation
   - Log as `mistake` in memory
   - Continue with remaining tests
```

### Steps

1. Implement feature following Technical Approach from Phase 1
2. Apply memory `preference` entries for code style
3. Run full test suite after each significant change
4. When all tests GREEN, run self-test checklist
5. Fix any checklist failures
6. Append to `FEATURE_LOG.md`: `Phase 3: Implementation complete — N/N tests GREEN`

**Exit Criteria:** All Phase 2 tests pass. Self-test checklist complete.

---

## Phase 4: Self-Review

**Goal:** Review implementation quality through 2 structured review rounds.

### Review Log Template

```markdown
## Feature Review: <slug>

### Round 1: Code Quality
Date: <ISO timestamp>
Passed: [Y/N]
Issues found:
- <list any: dead code, missing error handling, unclear naming>
Actions taken: <what was fixed>

### Round 2: Acceptance Criteria Coverage
Date: <ISO timestamp>
Passed: [Y/N]
Issues found:
- <any criteria not fully addressed>
Actions taken: <what was fixed>

### Final Verdict: PASS / FAIL
Reason: <brief summary>
```

### Review Checklist

**Round 1 — Code Quality:**
- [ ] All functions/methods have docstrings or comments where non-obvious
- [ ] No magic numbers — constants are named
- [ ] Error messages are human-readable
- [ ] No duplicate logic (DRY)
- [ ] Imports/dependencies are minimal and justified

**Round 2 — Acceptance Criteria Coverage:**
- [ ] Every Acceptance Criterion from Phase 1 has at least one passing test
- [ ] Edge cases documented in Phase 2 are handled
- [ ] Feature Spec Scope Boundary is respected (nothing out-of-scope added)
- [ ] No regressions (existing tests still pass)

### Steps

1. Run Round 1 code quality review; fix all issues found
2. Run Round 2 acceptance criteria review; fix any gaps
3. If Final Verdict is FAIL: fix issues, re-run both rounds (max 1 retry)
4. Append review summary to `FEATURE_LOG.md`
5. Run full test suite one final time — record result

**Exit Criteria:** Both review rounds complete, Final Verdict = PASS, all tests GREEN.

---

## Phase 5: Ship

**Goal:** Commit implementation and open pull request.

### Commit Message Convention

```
feat(<scope>): <one-line description>

<2-3 lines of context: what changed and why>

Closes #<issue> (if applicable)
Acceptance criteria: <brief list>
Tests: <N> new tests, all GREEN

Co-authored-by: <agent-name>
```

### PR Description Template

```markdown
## Summary
<2-3 sentences: what this PR does>

## Changes
- `<file>`: <what changed>
- `<file>`: <what changed>

## Testing
- Test file: `tests/test_<slug>.*`
- All N tests pass
- Edge cases covered: <list>
- No regressions detected

## Acceptance Criteria
- [x] <criterion 1>
- [x] <criterion 2>
...

## Notes for Reviewer
<any non-obvious design decisions or tradeoffs>

---
*Feature developed autonomously by /feature-dev pipeline*
```

### Copilot Path (MCP tools available)

```
1. github-push_files(branch="feat/<slug>", files=[all changed files], message=<commit msg>)
2. github-create_pull_request(
     owner=<user>,
     repo=<repo>,
     title="feat(<scope>): <description>",
     head="feat/<slug>",
     base=<base-branch>,
     body=<PR description>,
     draft=false
   )
3. Log PR URL to FEATURE_LOG.md and memory
```

### Cursor / git CLI Path

```bash
git add .
git commit -m "feat(<scope>): <description>"
git push origin feat/<slug>
gh pr create \
  --title "feat(<scope>): <description>" \
  --body "<PR description>" \
  --base <base-branch>
```

If `gh` is not available:
```bash
git push origin feat/<slug>
# Log: "Push complete. Create PR manually at: <repo>/compare/feat/<slug>"
```

### Steps

1. Detect platform from Phase 0 session state
2. Stage and commit all changed files with structured commit message
3. Push feature branch to remote
4. Create PR using appropriate path above
5. Log PR URL to `FEATURE_LOG.md`
6. Update `FEATURE_LOG.md`: `Phase 5: PR opened at <url>`

**Exit Criteria:** Commits pushed, PR created (or push confirmed if no `gh` CLI), PR URL logged.

---

## Phase 6: Memory Save

**Goal:** Persist feature development insights and update project memory.

### What to Save

Always save (minimum 3 entries):
1. **Feature fact**: `{type: feature, entity: project, fact: "Implemented '<slug>': <summary>. PR: <url>"}`
2. **Decision pattern**: Most significant technical decision as `{type: pattern, entity: project, ...}`
3. **Process learning**: What went well or poorly as `{type: mistake | pattern, entity: user, ...}`

Additionally save:
- Codebase pattern discovered during exploration
- Test framework preference if newly detected

### Feature Memory Entry Schema

```json
{
  "id": "uuid-v4",
  "type": "feature",
  "entity": "project",
  "fact": "feature-dev '<slug>': <summary>. Branch: feat/<slug>. PR: <url>",
  "confidence": 1.0,
  "source": "feature-dev-phase-6",
  "created_at": "2026-01-01T00:00:00Z",
  "status": "active"
}
```

### Steps

1. Construct ≥ 3 JSONL entries (one per line, no wrapping array)
2. Append to `user-memory.jsonl` at platform path
3. Check file size: if > 200 entries, archive oldest 100 to `*-archive.jsonl`
4. Finalize `FEATURE_LOG.md` with completion timestamp and memory update count
5. Print completion summary

**Exit Criteria:** ≥ 3 new entries appended to user-memory.jsonl. FEATURE_LOG.md complete.

---

## Error Recovery Table

| Failure | Strategy |
|---------|----------|
| GitHub API rate limit | Wait 60s, retry 3× with exponential backoff |
| Branch already exists (exact name) | Append `-v2` to slug, retry |
| Tests cannot be made GREEN (after 3 attempts) | Document in PR as known limitation, ship anyway with note |
| Implementation causes regressions | Revert last change, take narrower approach, document in spec |
| `gh` CLI not available | Push branch only; log manual PR creation URL |
| Codebase exploration fails (no file access) | Use feature description + memory to infer structure |
| Feature description too vague (< 3 words) | Expand to 2 interpretations, choose simplest, document in spec |
| Platform detection fails | Default to `local`, skip GitHub operations, produce local diff |
| Memory file corrupted | Rename `.corrupted.<ts>`, create new, log |

---

## Quality Gates

| Phase | Gate | Type |
|-------|------|------|
| 0 — Setup | Feature branch created, memory loaded | Hard |
| 1 — Spec | All 5 sections present, each ≥ 2 sentences | Hard |
| 2 — TDD | ≥ (N acceptance criteria + 3) tests, all RED | Hard |
| 3 — Implementation | All Phase 2 tests GREEN, self-test checklist passes | Hard |
| 4 — Review | Both review rounds PASS, no regressions | Hard (1 retry) |
| 5 — Ship | Commit pushed, PR URL logged | Hard |
| 6 — Memory | ≥ 3 new entries saved | Hard |

---

## Platform Compatibility

| Platform | Install Location | Activation | Repo Ops |
|----------|-----------------|------------|----------|
| GitHub Copilot (VS Code) | `.github/copilot/skills/feature-dev/SKILL.md` | `/feature-dev <desc>` | GitHub MCP tools |
| Cursor | `.cursor/rules/feature-dev.mdc` | `/feature-dev <desc>` | git CLI |
| Claude Code | `~/.claude/skills/feature-dev/SKILL.md` | `/feature-dev <desc>` | git CLI |
| Global (Copilot) | `~/.github/skills/feature-dev/SKILL.md` | `/feature-dev <desc>` | GitHub MCP tools |
| Global (Cursor) | `~/.cursor/skills/feature-dev/SKILL.md` | `/feature-dev <desc>` | git CLI |

### Install Commands

```bash
# GitHub Copilot (VS Code) — project scope
mkdir -p .github/copilot/skills/feature-dev
curl -sL https://raw.githubusercontent.com/codes1gn/feature-dev/main/SKILL.md \
  -o .github/copilot/skills/feature-dev/SKILL.md

# GitHub Copilot — global scope
mkdir -p ~/.github/skills/feature-dev
curl -sL https://raw.githubusercontent.com/codes1gn/feature-dev/main/SKILL.md \
  -o ~/.github/skills/feature-dev/SKILL.md

# Cursor — project scope
mkdir -p .cursor/rules
curl -sL https://raw.githubusercontent.com/codes1gn/feature-dev/main/SKILL.md \
  -o .cursor/rules/feature-dev.mdc

# Cursor — global scope
mkdir -p ~/.cursor/skills/feature-dev
curl -sL https://raw.githubusercontent.com/codes1gn/feature-dev/main/SKILL.md \
  -o ~/.cursor/skills/feature-dev/SKILL.md
```

---

## Harness Tool

A standalone test runner validates `/feature-dev` pattern compliance across the SKILL.md:

```bash
# Run test suite (fast smoke test)
python tests/run_tests.py

# Run with parallelism
python tests/run_tests.py --workers 4 --runs 5

# Full batch
python tests/run_tests.py --workers 8 --runs 10
```

The test suite checks 8 scenario groups (~7 checks each = ~56 checks/run) covering:
activation syntax, branching strategy, TDD workflow, Copilot PR path, Cursor PR path,
memory integration, spec template completeness, and review rounds.

See `tests/run_tests.py` for the full implementation.

---

## Comparison: /incubate vs /feature-dev

| Dimension | `/incubate` | `/feature-dev` |
|-----------|-------------|---------------|
| Starting point | Blank idea | Existing codebase |
| Output | New GitHub repo + README + website | PR to existing repo |
| Phases | 11 | 6 |
| TDD | Optional | Mandatory |
| Memory | Creates + updates | Reads + updates |
| Branching | N/A (new repo) | `feat/<slug>` branch |
| Best for | New side projects | Adding features to existing projects |
| Run time | 20–40 min | 5–15 min |

---
> Source: [codes1gn/feature-dev](https://github.com/codes1gn/feature-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
