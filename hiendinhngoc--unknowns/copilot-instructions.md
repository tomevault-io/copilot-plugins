## unknowns

> This repo publishes the `unknowns` Claude Code plugin and portable Agent Skills.

# CLAUDE.md

This repo publishes the `unknowns` Claude Code plugin and portable Agent Skills.

## Repo shape

Skills live flat under `skills/<name>/SKILL.md`:

- `blindspot` — read-only pre-implementation risk scan
- `verify-ref` — comprehension gate before porting/reference-copying
- `mock` — disposable prototype for unresolved UX/behavior
- `log-deviation` — record plan-vs-code deviations during implementation
- `merge-quiz` — pre-merge quiz on risky diff areas
- `unknowns` — router/orchestrator; keep routing logic here, technique logic in the target skill

Keep every skill referenced in the top-level `README.md` skills table. Plugin package metadata lives in `.claude-plugin/`; do not add per-skill entries there unless Claude's plugin format starts requiring them.

## Skill conventions

Each skill is one directory containing `SKILL.md` and `agents/openai.yaml` with:

```markdown
---
name: <kebab-name>
description: <trigger phrases + when to use it>
version: 1.0.0
author: Hien Dinh
license: MIT
---

# Title
```

Rules:

- `name` must match the directory.
- `description` drives model invocation; include user phrases people will actually say.
- Put hard constraints near the top with `<HARD-RULE>` or `<HARD-GATE>`.
- End with an explicit output format.
- If a skill mentions Claude-specific tools (`Artifact tool`, `AskUserQuestion`, `Skill tool`, `Glob/Grep/Read`), include the portable fallback in the same skill.
- Keep skills portable: no repo-specific paths, no hidden Claude-only requirement unless a fallback is documented.
- Keep `SKILL.md` under 500 lines and add resources only when progressive disclosure or deterministic tooling earns them.
- Keep `agents/openai.yaml` display metadata and `$skill-name` default prompt aligned with `SKILL.md`.

## Adding or changing skills

1. Update `skills/<name>/SKILL.md`.
2. Update the top-level `README.md` skills table and examples if the user-facing behavior changes.
3. If `unknowns` should route to the skill, update `skills/unknowns/SKILL.md`.
4. Run:

```bash
python3 scripts/validate_skills.py
python3 -m unittest discover -s tests -p 'test_*.py'
```

`tests/eval.sh` is a manual live-agent smoke test. Do not run it for routine docs/metadata edits unless the change affects real skill behavior.

## Pull requests

One focused skill change per PR. In the PR body, say what unknown the change surfaces and how it was checked.

## Commit messages

Use Conventional Commit prefixes on the first line so `release-please` can
detect releases, then keep Lore reasoning in the body and trailers.

Examples:

- `fix: make unknown-finding skills fail closed`
- `feat: add a skill for architecture blindspot reviews`
- `feat!: rename a public skill or change its invocation contract`

Keep the Lore trailer structure after the subject line; the prefix is the only
extra requirement.

---
> Source: [hiendinhngoc/unknowns](https://github.com/hiendinhngoc/unknowns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
