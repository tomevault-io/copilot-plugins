## lgtm

> {CONFIGURE: Initiative goal — brief description of the overarching goal}

# {CONFIGURE: Initiative name}

## Initiative Description

{CONFIGURE: Initiative goal — brief description of the overarching goal}

## Tools & Dependencies

{CONFIGURE: Key tools, packages, APIs, or services this initiative depends on}

## Commands

{CONFIGURE: Frequently used commands (build, generate, validate, deploy, etc.)}

## Hierarchy

Work flows through a structured hierarchy:
Initiative → Project → Milestone → Issue → Commit.

| Level | Filesystem | GitHub |
|-------|-----------|--------|
| Initiative | The repo | — |
| Project | `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` | GitHub milestone |
| Milestone | Section in PLAN.md or sub-plan | Encoded in issue ID prefix |
| Issue | `.lgtm/shared/plans/{project}/issues/P{N}M{M}-{NNN}.md` | PR (issue ID in title) |

The issue-to-PR contract: PR title starts with the issue ID
(e.g., `P2M1-001: short description`). Branch names are descriptive.
Each project gets a `{project}-base` branch; issue branches target it.

## Project Structure

```
.lgtm/                                 — workspace (plans, artifacts, status)
  STATUS.md                            — current initiative status
  shared/
    plans/
      p{N}-{slug}/
        PLAN.md                        — project plan (milestones, DAG, context)
        m{N}-{name}.md                 — milestone sub-plan (if scope warrants)
        issues/
          P{N}M{M}-{NNN}.md           — issue spec (agent or human)
  ai/
    process-artifacts/                 — all agent output, mirroring hierarchy
```

## Workflow

### Plan → Issue → PR lifecycle

1. **Plan** — define milestones and their dependency DAG
2. **Issues** — break the plan into atomic units of work with acceptance criteria
3. **Execute** — each issue = one branch + one PR targeting the project base branch
4. **Review** — quality check against acceptance criteria before PR
5. **Merge** — user reviews and merges each PR
6. **Status** — update `.lgtm/STATUS.md` after each merge, dispatch newly-ready issues

### Branch structure

```
master (production)
  └── develop (integration/testing)
        └── {project}-base (project isolation)
              ├── {project}-feature-a
              ├── {project}-feature-b
              └── {project}-feature-c
```

### Issue ID format

`P{project}M{milestone}-{sequence}` (e.g., `P2M1-003`)

## Conventions

- PRs only — never commit directly to master or develop
- One branch per issue, one PR per issue
- PR title is the issue-to-PR contract: `P{N}M{M}-{NNN}: short description`
- No AI attribution in commits (no Co-Authored-By trailers)
- Never stage all files at once — stage specific files only
- Commit messages explain WHY (the diff shows what)
- Dates use human-friendly format: "Apr 5, 2026" in tables
- Use YYYY-MM-DD for folder names

## Projects

Plans: `.lgtm/shared/plans/{project}/PLAN.md`
Status: `.lgtm/STATUS.md` (read on demand, not auto-loaded)

---
> Source: [syedsaif666/LGTM](https://github.com/syedsaif666/LGTM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
