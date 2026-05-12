## python-ai-guardrails-template

> A **PostToolUse hook** in `.claude/settings.json` runs `make check` automatically

# Claude Code — Quality Contract

## How the quality gate works

A **PostToolUse hook** in `.claude/settings.json` runs `make check` automatically
every time you edit or create a `.py` file. You do not need to run it manually —
the hook handles it.

If the hook **fails**, you must fix the issue before moving on. Do not proceed to
the next task. Do not mark a task complete. Until the check passes, keep fixing.

---

## Fixing failures

When `make check` fails:

1. Read the full output — every line
2. Run `make fix FILE=<filepath>` first (fixes formatting and safe ruff violations automatically)
3. Wait for the hook to re-run on the next edit, or manually run `make check FILE=<filepath>`
4. If still failing, fix the remaining issues manually
5. Keep editing until the check passes

Never ask the user whether to fix quality issues. Just fix them.

---

## Critical functions

Before editing any function, check `CRITICALITY.md`.

If the function is listed there:

- Type annotations are **mandatory** — full input and return types
- Docstring is **mandatory** — document contract, parameters, return value, and exceptions
- Existing tests must still pass after your change
- New behavior must have new tests

Raise extra scrutiny on high fan-in functions. Many callers means a subtle
behavior change can break things in non-obvious ways.

---

## Code style rules

- Prefer early returns over nested conditionals
- Extract helper functions rather than writing inline logic blocks
- Keep functions under 30 lines of actual logic
- Keep files under 300 SLOC — split at natural module boundaries
- Name things for what they *are*, not what they *do* (nouns for data, verbs for actions)
- No commented-out code — delete it, git has history

---

## Type annotation rules

When a type annotation exceeds the complexity thresholds enforced by
`scripts/check_type_complexity.py` (nesting depth > 2 or length > 40 chars),
replace it with a named type using the following decision rules.

### When to use each approach

**@dataclass:**

- Nesting depth > 2
- Same dict shape appears in more than one place in the codebase
- The dict has implied keys that are never documented
- A reader cannot understand what the data means from the type alone

**TypedDict:**

- Data comes from or goes to JSON/dict-based external APIs
- You cannot control construction (data arrives as a raw dict)
- You need structural compatibility with dict

**TypeAlias:**

- The nested type is genuinely primitive and used in exactly one place
- Example: `type TagMap = dict[str, list[str]]`  # shallow and obvious

### Decision flowchart

    Is the data coming from an external dict/JSON API?
    ├── yes → TypedDict
    └── no  → Is this shape used in more than one place?
              ├── yes → @dataclass
              └── no  → Is nesting depth > 2?
                        ├── yes → @dataclass
                        └── no  → TypeAlias is acceptable

### Hard rules

- Never write `dict[str, list[dict[str, str]]]` or deeper without a name
- Never use `# type: ignore` to suppress a complex annotation error
- When `check_type_complexity.py` fires, rethink the data shape — do not
  just rewrite the annotation to be shorter

---

## What you must never do

- Never silence mypy errors with `# type: ignore` without a comment explaining why
- Never use `# noqa` without a comment explaining why
- Never increase cyclomatic complexity without first asking if a refactor is possible
- Never reduce test coverage — 80% is a floor, not a target

---

## Security rules

- Never use `eval()`, `exec()`, or `pickle.loads()` on untrusted input
- Never hardcode credentials — use environment variables
- Never suppress bandit findings with `# nosec` without a comment explaining why
- Never suppress detect-secrets findings without updating `.secrets.baseline`
  and leaving a comment
- Run `make pip-audit` after adding any new dependency before continuing work
- If pip-audit reports a CVE, do not proceed — report it to the user immediately

---

## Running the full audit

When starting work on an unfamiliar part of the codebase, run:

```bash
make audit
```

This regenerates `CRITICALITY.md` and tells you which functions are
load-bearing before you touch anything.

---
> Source: [tortastudios/python-ai-guardrails-template](https://github.com/tortastudios/python-ai-guardrails-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
