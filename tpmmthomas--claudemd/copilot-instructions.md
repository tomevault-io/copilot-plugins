## claudemd

> Make code correct, maintainable, and verified — and spend effort in proportion

# Claude Coding Agent Instructions

## Core philosophy

Make code correct, maintainable, and verified — and spend effort in proportion
to the change. Process exists to protect large or risky work, not to tax small
work. A one-line answer should take one line.

---

## Scale the process to the change

Decide which tier a task falls in before starting. When unsure, pick the lower
tier and escalate if the work turns out bigger than it looked.

| Tier | Examples | What's required |
|---|---|---|
| **Trivial** | Answer a question, give a command, rename variables, fix a typo, tweak a config value, explain existing code | Just do it. No memory reads, no notes, no docs, no STATUS. Reply in a sentence or two. |
| **Normal** | Bug fix, new function, small feature, single-module change | Understand → implement → test → short report. Update docs only if behaviour or the public API changed. |
| **Significant** | New feature, refactor touching >5 files, breaking change, new/moved directory, anything that changes how the repo is laid out | Full workflow below: change doc, README map, `STATUS.md`, memory note. |

**Only write or change tracked docs, `STATUS.md`, and memory when the change is
significant.** The same bar that governs markdown changelogs governs everything
else. Routine work leaves no paperwork behind.

---

## Workflow (normal and significant work)

1. **Understand** — read the relevant code and tests first.
2. **Locate** — decide where new files belong before creating them.
3. **Implement** — clean, focused code; one concern per function.
4. **Test** — write and run tests; fix real failures rather than papering over them.
5. **Document** — inline comments always; a change doc only if significant.
6. **Record** — `STATUS.md` and a memory note only if significant.
7. **Verify** — check the task is actually done before saying so.

For significant work only, start by reading `.claude/memory/practices.md` and
`STATUS.md` so you don't rediscover what a previous session recorded.

---

## Repository organization

### Keep the root clean

Don't create new files in the repo root unless they're one of: `README.md`,
`CLAUDE.md`, `STATUS.md`, `LICENSE`, `.gitignore`, `.env.example`, the language
manifest/lockfile, or a linter/CI config.

Everything else — scripts, notes, experiments, sample data, generated output —
goes in a subdirectory.

### Canonical layout

Follow the repo's existing convention if it has one. Otherwise:

```
repo/
├── README.md          # entry point + repository map
├── STATUS.md          # project state (significant changes only)
├── src/<package>/     # source
├── tests/             # mirrors src/ one-to-one
├── scripts/           # runnable operational scripts
├── docs/              # design docs, ADRs, change docs
├── config/            # config, templates, schemas
├── .claude/memory/    # gitignored agent memory
└── .scratch/          # gitignored throwaway work
```

### Choosing a location

1. Search for existing files doing a similar job and put yours next to them.
2. Check the repository map in the root `README.md`.
3. Match sibling naming (`payment_service.py`, not `paymentSvc.py`).
4. If nothing fits, don't invent a top-level directory — propose it in your
   response with a one-line rationale and prefer nesting under an existing one.

### Naming

Descriptive and durable. Never `new_`, `old_`, `final_`, `v2_`, `_updated`,
`_fixed`, `copy_`, or `temp_` in a committed filename — version control handles
versions.

### Temporary files

Exploratory scripts and debugging harnesses go in `.scratch/` (gitignored), and
are deleted or promoted to `scripts/` before you finish. Generated artifacts
are gitignored, never committed.

### Moving files

Update every reference in the same change: imports, config, CI, docs, README
map. Prefer moving over duplicating.

---

## README

The root `README.md` should answer, within one screen: what is this, how do I
run it, where do I start reading, where does new code go.

- **Map directories, not files.** Describe each directory's purpose and name one
  entry-point file. Exhaustive file listings go stale immediately.
- **Structural change ⇒ README update, same commit.** Adding, removing,
  renaming, or repurposing a directory means updating the map. Nothing else
  requires touching the README.
- Give a directory its own `README.md` only if it has many files and a
  non-obvious purpose.
- If you notice the README contradicting reality, fix it and flag it.

---

## Agent memory

`.claude/memory/` is gitignored and never committed — not even when a commit is
explicitly requested. Confirm `.gitignore` covers it before writing there.

```
.claude/memory/
├── practices.md      # verified environment + commands
├── index.md          # one row per task note, newest first
└── tasks/YYYY-MM-DD-<slug>.md
```

### practices.md

The one file worth maintaining routinely. Record only what you **verified in
this repo** — a command goes in after it ran successfully. Fix a wrong entry the
moment you hit it; a stale command here costs every future session.

Keep it under 60 lines: environment, a command table (install / test / lint /
format / types), key conventions, and gotchas. Tables and fragments, no prose.
Delete what no longer applies — it keeps no history.

### Task notes

Write a note **only when the task was significant, or when it ended blocked or
abandoned** with something worth knowing. Routine fixes don't get notes.

Cap ~30 lines. Record facts, not narration. Reference code by path and symbol
rather than pasting it. Never write secrets or customer data. Don't rewrite old
notes — add a `Superseded by:` line instead.

```markdown
# <Title>
Date: YYYY-MM-DD · Status: done | partial | blocked

## Goal
## Changes         — file → what changed
## Decisions       — chose X over Y because Z
## Verification    — command → result
## Gotchas / follow-ups
```

Then prepend one row to `index.md` (date, slug, one-line summary, path).

Memory is local and invisible to teammates, so it never substitutes for tracked
docs. Anything another person needs goes in `README.md`, `STATUS.md`, `docs/`,
or the code.

---

## STATUS.md

Tracked in git, written for a human, answering *where is this project right now*.

**Update it only for significant changes**, in the same commit. Bug fixes,
small features, and refactors that don't change the project's shape leave it
alone.

Hard cap ~50 lines. Tables over paragraphs. Summarize, don't accumulate: keep at
most 7 recent-change rows and fold older ones into a single `Earlier` row.
Describe the repository, not the session — no "I", no mention of agents.

```markdown
# Project Status
Updated: YYYY-MM-DD
**Health:** green | yellow | red — one-line reason
**Now:** what's actively being worked on

## Components        | Area | State | Notes |
## Recent changes    | Date | Change | Ref |   (max 7)
## Todo / next       | Priority | Item | Notes |
## Known issues      | Issue | Impact | Workaround |
```

Don't paste test output or duplicate the README map. Link tracked docs, not task
notes (they're gitignored and readers can't open them).

---

## Testing

- New behaviour gets tests. Every bug fix gets a regression test that would have
  caught it. Renames, comment changes, and config tweaks don't need new tests.
- Cover happy path, edge cases, and expected failures.
- Name tests as specifications: `test_raises_value_error_on_negative_index()`.
- Run the suite after changes and report pass/fail counts. Don't mark a task
  complete while it's red.
- If a pre-existing test fails because of your change, flag it explicitly and
  either fix the cause or explain why the behaviour change is intended.
- Tests live in `tests/`, mirroring the source tree: `src/app/core/pricing.py` →
  `tests/core/test_pricing.py`. Shared fixtures in `tests/conftest.py`, data in
  `tests/fixtures/`.
- Use the commands in `practices.md`; if you work out the right invocation the
  hard way, record it there.

---

## Documentation

### Inline comments — the default, and usually the only doc needed

- **Why, not what.** Explain intent, not what the code literally does.
- Comment non-obvious algorithms, regexes, and bit manipulation.
- Always explain workarounds, hacks, and TODOs with context.
- Public functions/classes get a docstring; new modules get a one-to-three line
  header saying what they're responsible for.

```python
# Good: We skip the first byte because the vendor prefixes payloads with 0xFF
data = raw[1:]
```

### Change docs — significant changes only

| Situation | File |
|---|---|
| New feature | `docs/features/<name>.md` |
| Refactor >5 files | `docs/changes/<YYYY-MM-DD>-<slug>.md` |
| Breaking API change | `BREAKING_CHANGES.md` (append) |
| Architecture decision | `docs/adr/<NNN>-<slug>.md` |
| New/moved/removed directory | Root `README.md` map |

Cover: what changed, why, how it works, where it lives, and any migration
impact. Link it from `docs/README.md`.

---

## Code quality

- Follow the codebase's existing style. Consistency beats preference.
- One thing per function. If describing it needs "and", split it.
- Prefer explicit over clever. Delete dead code rather than commenting it out.
- One responsibility per module; avoid catch-all `utils`/`helpers`/`common`.
  Name things by what they do (`retry.py`), not what they aren't.
- Dependencies flow one way — core domain logic doesn't import adapters or
  framework code.
- Never silently swallow exceptions. Validate inputs at the boundary. Use
  domain-specific error types.
- Don't add a dependency for something trivially done in stdlib. When you do add
  one, pin it and say so in your response.
- Never hard-code secrets. Sanitize external input. Mark security-relevant
  changes with a `# SECURITY:` comment.

---

## Git hygiene

- Atomic commits, imperative messages (`Add retry logic to HTTP client`).
- No commented-out code, debug prints, scratch files, or generated output.
- Never commit `.claude/memory/`.
- Separate pure file moves from content edits where practical.

---

## Before saying done

For **trivial** work: just make sure the answer is right.

For **normal** work:
- [ ] Tests pass; new behaviour and bug fixes are covered.
- [ ] Linter/formatter passes.
- [ ] No new files in the repo root; new files sit where the map says.
- [ ] Tests mirror the source tree; no `temp_`/`v2_` filenames.
- [ ] `.scratch/` is clean; new public symbols have docstrings.
- [ ] No secrets, dead code, or debug statements left behind.

For **significant** work, additionally:
- [ ] README map matches the current structure.
- [ ] Change doc written and linked from `docs/README.md`.
- [ ] `STATUS.md` updated, within cap, old rows folded not appended.
- [ ] Task note written and indexed; `practices.md` corrected if anything there
      turned out wrong.
- [ ] `.claude/memory/` still ignored and untracked.

---

## Response format

Match the report to the tier.

**Trivial:** answer directly. No summary block, no file listing, no headers.

**Normal:** a few lines — what changed, which files, test results.

**Significant:**

```
### Done
**What I did:** ...
**Files:**
  + src/app/core/pricing.py      new — domain pricing rules
  ~ README.md                    updated repository map
  - scripts/old_migrate.py       removed, superseded by ...
**Tests:** X passed, 0 failed
**Docs:** <path-to-change-doc>.md / README map updated
**Memory & status:** note `<file>.md` · practices.md: <what changed> ·
STATUS.md: <what changed>
**Notes:** caveats, follow-ups, new dependencies, proposed new top-level
directories, or a README found out of date.
```

If you can't justify a file's location in one line, it's in the wrong place.
Memory files are untracked and don't appear in the Files block.

---
> Source: [tpmmthomas/claudemd](https://github.com/tpmmthomas/claudemd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
