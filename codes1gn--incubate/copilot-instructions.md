## incubate

> /incubate — Fully autonomous project incubation pipeline. Takes an idea through PRD → Architecture → Development → Testing → README → Website with persistent cross-session memory.


# /incubate — Autonomous Project Incubation Pipeline

## Overview

`/incubate <idea>` launches a fully autonomous 11-phase pipeline that takes any project idea from raw concept to a shipped GitHub repo with README and GitHub Pages website — **without requiring any user interaction during execution**.

Designed for solo developers who want to spin up side projects on weekends using compute power as a force multiplier. Run best-of-N instances in parallel and choose the best output.

## Activation

```
/incubate <idea>
/incubate <idea> --tag <tagname>
/resume <tagname>
```

Examples:
- `/incubate "CLI tool to batch-rename files with AI"`
- `/incubate "personal finance dashboard" --tag finance-dash`
- `/resume finance-dash`

## ⚠️ Autonomous Operation Declaration

**This skill operates in fully autonomous mode by design.**

- ❌ **Never** call `ask_user`, `AskQuestion`, `AskUserQuestion`, or any interactive prompt during Phases 1–11
- ✅ When ambiguous: interpret the idea three ways, choose the most actionable, document in Decision Log
- ✅ When blocked: apply the Error Recovery table, log as `mistake` in memory, continue
- ✅ Every significant decision is logged for best-of-N parallel-run comparison

The user is expected to be away. When they return, they review `STATUS.md` and `user-memory.jsonl`.

---

## Memory Protocol

### Storage Paths

| Platform | Memory Path |
|----------|-------------|
| GitHub Copilot (VS Code) | `~/.copilot/skills/incubate/data/user-memory.jsonl` |
| Cursor | `~/.cursor/skills/incubate/data/user-memory.jsonl` |
| Windows (Copilot) | `%USERPROFILE%\.copilot\skills\incubate\data\user-memory.jsonl` |
| Windows (Cursor) | `%USERPROFILE%\.cursor\skills\incubate\data\user-memory.jsonl` |

### Entry Schema

```json
{
  "id": "uuid-v4",
  "type": "preference | fact | pattern | mistake | decision",
  "entity": "user | project | tool",
  "fact": "human-readable fact statement",
  "confidence": 0.9,
  "source": "phase-N",
  "created_at": "2026-01-01T00:00:00Z",
  "status": "active"
}
```

### Memory Rules

- Load at Phase 0; apply `preference` and `pattern` entries throughout execution
- Save at Phase 11 (minimum 3 new entries)
- Archive when JSONL exceeds 200 entries (move oldest 100 to `*-archive.jsonl`)
- On corrupted file: rename to `.corrupted.<timestamp>`, create new file, log as `mistake`

### Platform Detection

| Signal | Platform | Repo Operations |
|--------|----------|-----------------|
| `github-create_repository` MCP tool available | Copilot (VS Code) | GitHub MCP tools |
| Tool unavailable + `git` in PATH | Cursor / Claude Code | git CLI |
| Neither available | Unknown | Local folder only |

---

## Phase 0: Memory Load & Setup

**Goal:** Initialize session state, load memory, detect platform, generate project slug.

### Steps

1. **Load Memory**
   - Check if `user-memory.jsonl` exists at the platform path
   - If exists: parse all `active` entries, build working context
   - If not exists: create directory structure, initialize empty file
   - If corrupted: rename to `<name>.corrupted.<timestamp>`, create fresh file

2. **Detect Platform**
   - Test if `github-create_repository` MCP tool is available → `PLATFORM=copilot`
   - Else test if `git --version` succeeds → `PLATFORM=cursor`
   - Else → `PLATFORM=local`
   - Log to `session.json`

3. **Generate Project Slug**
   - Normalize idea: lowercase, replace spaces with `-`, strip special chars, truncate to 40 chars
   - If `--tag` provided: use tag as slug override
   - Check if slug exists in memory (previous incubations):
     - If exists: append `-v2` (or `-v3` etc.) and continue
   - Store slug in session state

4. **Create Project Directory**
   - Local: `./incubations/<slug>/`
   - Initialize `session.json`: `{slug, idea, started_at, platform, phase: 0}`

**Exit Criteria:** Memory loaded, platform detected, slug set, directory created.

---

## Phase 1: PRD (Product Requirements Document)

**Goal:** Write a complete 8-section PRD without user input.

### PRD Template

```markdown
# PRD: <Project Name>

## 1. Problem Statement
What specific pain point does this solve? Who experiences it?

## 2. Target Users
Primary user persona(s). Be specific: "solo developer with 2–5 side projects."

## 3. Core Value Proposition
One sentence: "X helps Y do Z without W."

## 4. MVP Features
Bullet list of must-have features for v1. Max 7 items.

## 5. Non-Goals
Explicit exclusions to prevent scope creep.

## 6. Success Metrics
3–5 measurable outcomes. At least one quantitative metric.

## 7. Technical Constraints
Known constraints: language, platform, dependencies, auth requirements.

## 8. Autonomous Interpretation
(a) 3 possible interpretations of the idea
(b) Chosen interpretation
(c) Rationale for choice
```

### Steps

1. Analyze the idea using loaded memory (apply relevant `preference` and `pattern` entries)
2. Fill each section — minimum 2 sentences per section
3. For Section 8: always populate, even for clear ideas (document assumptions)
4. Save to `./incubations/<slug>/PRD.md`
5. Update `session.json` → `phase: 1, prd_complete: true`

**Exit Criteria:** PRD file exists, all 8 sections present, each ≥ 2 sentences.

---

## Phase 2: Self-Review

**Goal:** Validate PRD quality through 3 structured review rounds.

### Review Log Template

```markdown
# Review Log: <slug>

## Round 1: Completeness Check
Date: <ISO timestamp>
Passed: [Y/N]
Issues: <list any missing sections or thin content>
Actions taken: <what was fixed>

## Round 2: Technical Feasibility
Date: <ISO timestamp>
Passed: [Y/N]
Issues: <unrealistic metrics, missing constraints, vague tech stack>
Actions taken: <what was fixed>

## Round 3: Autonomous Operation
Date: <ISO timestamp>
Passed: [Y/N]
Issues: <anything requiring user input to resolve>
Actions taken: <what was fixed>

## Final Verdict: PASS / FAIL
Reason: <brief summary>
```

### Steps

1. **Round 1 — Completeness:** Check all 8 PRD sections exist and are substantive. Fix any gaps.
2. **Round 2 — Technical Feasibility:** Check metrics are measurable, constraints realistic, tech stack achievable.
3. **Round 3 — Autonomous Operation:** Ensure no section requires user input. Resolve all ambiguities.
4. Save Review Log to `./incubations/<slug>/REVIEW_LOG.md`
5. If Final Verdict is FAIL: update PRD, re-run all 3 rounds (max 1 retry)
6. Update `session.json` → `phase: 2, review_complete: true, review_verdict: PASS`

**Exit Criteria:** Review Log exists, all 3 rounds complete, Final Verdict = PASS.

---

## Phase 3: Architecture

**Goal:** Design the full technical architecture in 7 sections.

### Architecture Template

```markdown
# Architecture: <Project Name>

## 1. System Overview
One-paragraph summary with ASCII diagram or description.

## 2. Technology Stack
| Layer | Technology | Rationale |
|-------|------------|----------|

## 3. Data Model
Key data structures, schemas, or file formats.

## 4. Core Algorithms / Logic
Step-by-step description of the main processing logic.

## 5. Platform Compatibility Matrix
| Platform | Supported | Notes |
|----------|-----------|-------|
| macOS    | ✅        |       |
| Linux    | ✅        |       |
| Windows  | ✅/⚠️     |       |

## 6. Error Handling Strategy
Map of failure modes to recovery strategies.

## 7. Decision Log
| Decision | Options Considered | Chosen | Rationale | Memory Influence |
|----------|-------------------|--------|-----------|------------------|
```

### Steps

1. Choose technology stack based on PRD constraints + memory preferences
2. Design data model — prefer simple formats (JSON, JSONL, Markdown) unless PRD requires DB
3. Document all significant decisions in Decision Log
4. Save to `./incubations/<slug>/ARCHITECTURE.md`
5. Update `session.json` → `phase: 3, arch_complete: true`

**Exit Criteria:** Architecture file exists, all 7 sections present.

---

## Phase 4: Development

**Goal:** Implement the project, produce working code.

### Self-Test Checklist

Before shipping, verify:
- [ ] Main entry point exists and is documented
- [ ] All PRD MVP features implemented
- [ ] README stub present (will be enhanced in Phase 8)
- [ ] No hardcoded credentials or secrets
- [ ] Basic error handling present
- [ ] Works without modification on target platform(s)

### Steps

1. Scaffold project structure based on Architecture
2. Implement all MVP features from PRD Section 4
3. Apply memory `preference` entries (e.g., preferred language, style)
4. Run self-test checklist — fix all failures
5. Save code to `./incubations/<slug>/src/`
6. Update `session.json` → `phase: 4, dev_complete: true`

**Exit Criteria:** Self-test checklist passes, all MVP features present, no secrets in code.

---

## Phase 5: Ship

**Goal:** Create GitHub repository and push all project files.

### Copilot Path (MCP tools available)

```
1. github-create_repository(name=slug, description=..., private=false)
2. Collect all files from ./incubations/<slug>/
3. github-push_files(owner, repo=slug, branch=main, files=[...],
                     message="Initial incubation: <idea>")
4. Log repo URL to session.json and memory
```

Note: GitHub Pages must be enabled manually (Settings → Pages → Source: GitHub Actions).

### Cursor / git CLI Path

```bash
cd ./incubations/<slug>
git init && git add . && git commit -m "Initial incubation: <idea>"
gh repo create <slug> --public --push --source .
```

### Steps

1. Detect platform from Phase 0 session state
2. Execute appropriate path above
3. Verify: repo is accessible at `https://github.com/<user>/<slug>`
4. Log repo URL to `session.json` and memory (`type: fact, entity: project`)
5. Update `session.json` → `phase: 5, ship_complete: true, repo_url: <url>`

**Exit Criteria:** Repo visible on GitHub, all files pushed, URL in memory.

---

## Phase 6: Test Framework

**Goal:** Create a pattern-based test suite validating skill structure and behavior.

### Test Scenarios (20 required)

| ID | Scenario | What It Validates |
|----|----------|-------------------|
| 01 | `valid-idea-copilot` | Copilot platform path documented |
| 02 | `valid-idea-cursor` | Cursor platform path documented |
| 03 | `vague-idea` | Autonomous Interpretation handles vague ideas |
| 04 | `duplicate-slug` | Slug deduplication with `-v2` suffix |
| 05 | `memory-load` | Memory JSONL loads and applies correctly |
| 06 | `memory-save` | Phase 11 saves ≥ 3 new memory entries |
| 07 | `prd-completeness` | PRD template has all 8 required sections |
| 08 | `review-rounds` | Review Log has 3 rounds + Final Verdict |
| 09 | `arch-completeness` | Architecture has all 7 sections |
| 10 | `test-coverage` | Test suite has ≥ 13 scenarios |
| 11 | `batch-threshold` | Batch targets ≥ 1000 total checks |
| 12 | `readme-sections` | README has all 11 required sections |
| 13 | `website-valid` | docs/index.html valid with required elements |

### Steps

1. Create `tests/run_tests.py` with 13 scenario pattern checks
2. Create `tests/scenarios/` with one `.md` file per scenario
3. Run smoke test: `python tests/run_tests.py --workers 1 --runs 1`
4. Fix any failures
5. Update `session.json` → `phase: 6, tests_complete: true`

**Exit Criteria:** `run_tests.py` exists, 13 scenarios defined, smoke test passes.

---

## Phase 7: Batch Testing

**Goal:** Run ≥ 1000 total checks across parallel workers.

### Test Math

```
13 scenarios × avg 7 checks/scenario = 98 checks/run
98 checks/run × 8 workers × 10 runs = 7,840 total checks (target: ≥ 1000)
```

### Steps

1. Run: `python tests/run_tests.py --workers 8 --runs 10`
2. Collect results: pass rate, total checks, elapsed time
3. If pass rate < 95% after first run: identify failing scenarios, fix, retry (max 3 attempts)
4. If still < 95% after 3 attempts: ship with `SHIPPING_NOTES.md`, log as `mistake` in memory
5. Record final stats in `session.json`: `{total_checks, pass_rate, workers, runs, elapsed_seconds}`
6. Update `session.json` → `phase: 7, batch_complete: true`

**Exit Criteria:** ≥ 1000 total checks run, ≥ 95% pass rate (or documented exception).

---

## Phase 8: README

**Goal:** Write an attractive, comprehensive README (11 sections).

### 11-Section README Template

```markdown
# <project-name> — <tagline>
<!-- HTML-centered header + badges row + navigation links -->

## The Problem
<!-- 3-4 lines: what pain, who has it -->

## The Solution
<!-- 2-3 lines + one command example -->

## Quick Start
<!-- Shell block: install + run in ≤ 5 lines -->

## How It Works
<!-- Numbered steps or ASCII diagram -->

## Platform Support
<!-- Table: platform × install path × status -->

## Memory Layer
<!-- What gets remembered, where it lives -->

## Configuration
<!-- Key options, defaults, env vars -->

## Test Results
<!-- Badge or inline stats: X checks, Y% pass rate -->

## Contributing
<!-- One paragraph: PRs welcome, issue template -->

## License
<!-- SPDX identifier + one-line summary -->

## Acknowledgements
<!-- Prior art, inspiration, credits -->
```

### Steps

1. Write README.md following the 11-section template
2. Apply patterns from memory (e.g., `preference: uses badge shields`)
3. Include actual test stats from Phase 7
4. Save to `./incubations/<slug>/README.md`
5. Push to GitHub (update if existing)
6. Update `session.json` → `phase: 8, readme_complete: true`

**Exit Criteria:** README.md pushed to GitHub, all 11 sections present.

---

## Phase 9: Website

**Goal:** Build GitHub Pages landing page and configure CI deployment.

### docs/index.html Requirements

- Dark background (#0d1117 GitHub dark)
- Stats bar (total checks, pass rate, platforms supported)
- Tabbed install section (Copilot / Cursor / Claude)
- Before/After demo (problem vs solution)
- Call-to-action button → GitHub repo
- Fully self-contained (no external CDN required)

### .github/workflows/pages.yml

```yaml
name: Deploy GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: docs
      - id: deployment
        uses: actions/deploy-pages@v4
```

### Steps

1. Write `docs/index.html` (dark terminal aesthetic)
2. Write `.github/workflows/pages.yml`
3. Write `.github/workflows/tests.yml` (CI: runs test suite on push)
4. Push all three files to GitHub
5. Note: GitHub Pages requires manual activation (Settings → Pages → Source: GitHub Actions)
6. Update `session.json` → `phase: 9, website_complete: true`

**Exit Criteria:** docs/index.html + pages.yml + tests.yml in repo and pushed.

---

## Phase 10: Verify

**Goal:** Confirm everything is in place and produce a status report.

### Status Report Template

```markdown
# Incubation Status Report: <slug>

## Summary
- Idea: <original idea>
- Tag: <slug>
- Platform: <copilot|cursor|local>
- Started: <ISO timestamp>
- Completed: <ISO timestamp>

## Deliverables
- [ ] PRD: ./incubations/<slug>/PRD.md
- [ ] Review Log: ./incubations/<slug>/REVIEW_LOG.md
- [ ] Architecture: ./incubations/<slug>/ARCHITECTURE.md
- [ ] Source Code: ./incubations/<slug>/src/
- [ ] GitHub Repo: https://github.com/<user>/<slug>
- [ ] README: pushed to GitHub
- [ ] Test Results: <total_checks> checks, <pass_rate>% pass
- [ ] GitHub Pages: ⚠️ Requires manual activation

## Manual Actions Required
1. Enable GitHub Pages: <repo_url>/settings/pages → Source: GitHub Actions

## Decision Log Summary
<3–5 most impactful decisions from Phase 3 Decision Log>

## Memory Updates (Phase 11 preview)
<list of entries to be saved>
```

### Steps

1. Check GitHub repo exists and is public
2. Check main branch has all expected files
3. Check GitHub Actions ran (may show as queued if pushed recently)
4. Generate Status Report → save to `./incubations/<slug>/STATUS.md`
5. Update `session.json` → `phase: 10, verify_complete: true`

**Exit Criteria:** Status report written, all deliverables verified.

---

## Phase 11: Memory Save

**Goal:** Persist at least 3 new memory entries from this incubation.

### What to Save

Always save (minimum 3):
1. **Project fact**: `{type: fact, entity: project, fact: "Incubated '<slug>': <summary>. Repo: <url>"}`
2. **Decision pattern**: Most significant architectural decision as a `pattern` entry
3. **Process learning**: What went well or badly as a `mistake` or `pattern` entry

Additionally save:
- User preferences revealed by idea characteristics
- Notable technical choices for future reference

### Steps

1. Construct ≥ 3 JSONL entries (one per line, no wrapping array)
2. Append to `user-memory.jsonl` at platform path
3. Create directory if it doesn't exist
4. Check file size: if > 200 entries, archive oldest 100 to `*-archive.jsonl`
5. Log: `Memory updated: N new entries (total: M)`
6. Update `session.json` → `phase: 11, memory_saved: true, new_entries: N`

**Exit Criteria:** ≥ 3 new entries appended to user-memory.jsonl.

---

## Error Recovery Table

| Failure | Strategy |
|---------|----------|
| GitHub API rate limit | Wait 60s, retry 3× with exponential backoff |
| Repo already exists | Use existing repo, push as update |
| Test pass rate < 95% after 3 attempts | Ship with `SHIPPING_NOTES.md`, log as `mistake` |
| Memory file corrupted | Rename `.corrupted.<ts>`, create new, log |
| Idea too vague (< 5 words) | Expand to 3 interpretations, choose most actionable |
| Existing project same slug | Add `-v2` suffix (then `-v3` etc.), continue |
| Platform detection fails | Default to `local`, continue without GitHub push |
| Phase fails quality gate | Log issue, fix, retry once, then continue with note |

---

## Quality Gates

| Phase | Gate | Type |
|-------|------|------|
| 1 — PRD | All 8 sections present, each ≥ 2 sentences | Hard |
| 2 — Review | All 3 rounds complete, Final Verdict = PASS | Hard (1 retry) |
| 3 — Architecture | All 7 sections present | Hard |
| 4 — Development | Self-test checklist passes | Hard |
| 5 — Ship | Repo visible on GitHub | Hard |
| 6 — Tests | 13 scenarios, smoke test passes | Hard |
| 7 — Batch | ≥ 1000 checks, ≥ 95% pass rate | Soft (exception allowed) |
| 8 — README | All 11 sections present | Hard |
| 9 — Website | docs/index.html valid, pages.yml present | Hard |
| 11 — Memory | ≥ 3 new entries saved | Hard |

---

## Platform Compatibility

| Platform | Install Location | Memory Path | Repo Ops |
|----------|-----------------|-------------|----------|
| GitHub Copilot (VS Code) | `~/.github/skills/incubate/` | `~/.copilot/skills/incubate/data/` | MCP tools |
| Cursor | `~/.cursor/rules/` | `~/.cursor/skills/incubate/data/` | git CLI |
| Claude Code | `~/CLAUDE.md` (global) | `~/.claude/skills/incubate/data/` | git CLI |
| Windows (any) | `%USERPROFILE%\.<platform>\...` | Same with backslash | Same |

---

## /resume Protocol

When `/resume <tagname>` is invoked:

1. Load `session.json` from `./incubations/<tagname>/`
2. Read current `phase` field
3. Load `user-memory.jsonl`
4. Jump to the phase after the last completed phase
5. Continue autonomously from that phase

If `session.json` does not exist: treat as fresh `/incubate <tagname>`.

---
> Source: [codes1gn/incubate](https://github.com/codes1gn/incubate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
