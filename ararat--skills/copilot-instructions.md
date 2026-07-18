## skills

> This repo is a skill library. Whether you are *using* these skills in another

# Agent instructions

This repo is a skill library. Whether you are *using* these skills in another
project or *editing* this repo itself, the rules below apply.

## Using the skills

- **Router first.** Unsure which skill fits → read `engineering/navigate/SKILL.md`.
- **Precondition.** The tracker-touching skills (`to-prd`, `to-issues`, `triage`,
  `code-review`, `wayfinder`) assume `docs/agents/issue-tracker.md` exists in the
  target repo. Missing → run `/setup-skills` before them.
- **Vocabulary is load-bearing.** Skills use exact terms (seam, depth, tracer
  bullet, frontier, grounded). Don't paraphrase them into synonyms — consistent
  language is how the skills stay predictable.
- **Search learnings before diagnosing.** If the target repo has
  `docs/solutions/`, grep it for the symptom before starting `/diagnosing-bugs`.
- **Capture after hard-won work.** Finish anything non-obvious → offer
  `/compound-learnings`.

## Editing this repo

- Follow `productivity/writing-great-skills/SKILL.md` — it is the house style.
  Prune no-ops; keep one meaning in one place; prefer a leading word over a
  restatement; every step ends on a checkable completion criterion.
- Frontmatter: `name` must equal the folder name; `description` is trigger-rich
  for model-invoked skills, one human-facing line for user-invoked ones.
- Reference files only where a branch genuinely needs them; otherwise inline.
- Before committing: run `scripts/skill-lint.sh` — it must print `OK`.
- Attribution in `NOTICE.md` and `LICENSE` is preserved verbatim; never remove
  upstream copyright notices.

---
> Source: [ararat/skills](https://github.com/ararat/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
