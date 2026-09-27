## agentsmd-review

> Guidance for AI agents and contributors working in this repository.

# AGENTS.md

Guidance for AI agents and contributors working in this repository.

## What this project is

`agentsmd-review` compares two versions of an `AGENTS.md` file at the level of
rules, not raw text. It parses each version into normative directives (must,
must not, should, never, may), pairs them across versions, and reports what was
added, removed, weakened, strengthened, reversed, or reworded. Changes touching
a protected topic (secrets, force-push, production data, and so on) are
escalated, and the whole diff is graded A-F so a CI job can block a risky edit.

## Layout

- `agentsmd_review/parse.py` — Markdown to directives (modal detection, action
  normalisation).
- `agentsmd_review/diff.py` — pairing and change classification, protected-term
  escalation.
- `agentsmd_review/grade.py` — change-risk score, A-F grade, and gates.
- `agentsmd_review/report.py` — console / json / markdown / badge output.
- `agentsmd_review/cli.py` — argument parsing, file/demo/git modes, exit codes.
- `agentsmd_review/samples/` — before/after sample files used by the demo and
  tests.
- `tests/` — `unittest` suite.

## Hard constraints

- You must keep this project standard-library only, with no third-party runtime
  dependencies.
- You must not use syntax newer than Python 3.9.
- Every new behaviour should ship with a test, ideally exercised through the
  sample files.

## Dev commands

```bash
python -m unittest discover -s tests -v
python -m agentsmd_review --demo
python -m agentsmd_review --demo --format markdown
```

## Conventions

- New change types or severities go through `diff.py`; keep the severity words
  to critical / high / medium / low so the grader stays consistent.
- Reserve `critical` for a protected directive being removed, weakened, or
  reversed.
- When you add a protected topic, add it to `DEFAULT_PROTECTED` and cover it
  with a test.

---
> Source: [shriramkv/agentsmd-review](https://github.com/shriramkv/agentsmd-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
