## skills-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A version-controlled collection of **Agent Skills** (the open `agents.md`
`SKILL.md` format), reusable across multiple coding agents (Claude Code, OpenAI
Codex, Cursor, Windsurf). There is no build, test, or lint step — this is a
documentation/config repo, not a codebase with source to compile. The identity
is deliberate: *"bring discipline to vibe coding"* — skills that harden the
fast, LLM-generated code the author writes day to day.

## Layout

```
skills-hub/
├── README.md                     # Korean (primary)
├── README.en.md                  # English
├── LICENSE                       # MIT
├── CHANGELOG.md                  # per-release notes; tag releases in git
└── skills/
    └── <skill-name>/             # one folder per skill
        ├── SKILL.md              # required: name + description frontmatter
        └── references/           # optional docs loaded on demand
```

Each skill is self-contained under `skills/<name>/`. Adding a skill = adding a
folder — there is no registry file to update elsewhere.

## Install / distribution

Skills are installed via the **[skills.sh](https://skills.sh) CLI**
(`vercel-labs/skills`), NOT a custom script:

```bash
npx skills add sjsylee/skills-hub
```

The CLI auto-discovers `skills/<name>/SKILL.md` (flat layout) directly from the
GitHub repo — no clone, no manifest, no registry file required. Keeping the
`skills/<name>/SKILL.md` structure intact is what makes this work; don't nest
skills deeper without checking the CLI's discovery rules.

> There is intentionally **no `install.sh`** — it was removed as redundant with
> the CLI. Don't reintroduce a custom installer unless the CLI can't cover a
> real need.

## Authoring a new skill

- `SKILL.md` frontmatter must contain only `name` and `description` — keep to
  that shared standard so the skill stays cross-agent compatible. Put Codex-only
  UI metadata in an optional `agents/openai.yaml` sibling, which other agents
  ignore.
- The `description` is what triggers the skill; write it with concrete trigger
  phrases/synonyms a user might type, not just an abstract summary (see
  `skills/codebase-security-audit/SKILL.md` for the pattern).
- Put heavier reference material (checklists, per-stack notes) in
  `references/*.md` and point to them from `SKILL.md`, rather than inlining
  everything.
- Record the change in `CHANGELOG.md` under `[Unreleased]`.
- Keep both READMEs in sync: `README.md` (Korean) is primary, `README.en.md`
  mirrors it in English. Update the "Skills" list in both when adding a skill.

## Skill design conventions (established by `codebase-security-audit`)

Skills that produce a report or take consequential action follow a
**report-first, consent-gated** pattern: never batch-edit as a side effect of
analysis. Deliver findings/a plan first, wait for explicit user agreement, then
apply changes one at a time. Follow this when authoring or editing skills that
could otherwise be tempted to "just fix it" automatically.

## Versioning

This repo is the source of truth; version it with git.

- Tag releases: `git tag v1.0.0 && git push --tags`.
- Record changes per release in `CHANGELOG.md`.
- Users update by re-running `npx skills add sjsylee/skills-hub` (the CLI copies
  skills in; it does not symlink).

---
> Source: [sjsylee/skills-hub](https://github.com/sjsylee/skills-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
