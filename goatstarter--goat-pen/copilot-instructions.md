## goat-pen

> Rules for humans and agents working inside `goat-pen`. This pack follows

# AGENTS.md

Rules for humans and agents working inside `goat-pen`. This pack follows
**Goat Pack Standard v1**.

## What this repo is

x

Skills are markdown files that give a coding agent a method for a specific task. They
follow the [Agent Skills specification](https://agentskills.io/specification.md), so they
work in Claude Code, Codex, Cursor, and anything else that reads `SKILL.md`.

## Layout

```
goat-pen/
├── .claude-plugin/          # Claude Code plugin + marketplace manifests
├── skills/<name>/
│   ├── SKILL.md             # required, under 500 lines
│   ├── references/          # optional detail, loaded on demand
│   ├── evals/evals.json     # required, minimum 3 cases
│   └── scripts/             # optional, zero dependency
├── AGENTS.md                # this file
├── VERSIONS.md              # changelog + per-skill version table
├── install.sh               # copies skills into a target project
└── validate.sh              # structural check, run before every commit
```

## Before you commit

```sh
./validate.sh
```

It must exit 0. It checks frontmatter, name-to-directory match, Turkish triggers,
line limits, the Definition of done section, eval count, manifest version agreement,
and that `VERSIONS.md` matches every `SKILL.md`.

## Frontmatter

```yaml
---
name: skill-name
description: <what it does>. Use when the user <situation>. Also use when the user says "<phrase>", "<phrase>". Türkçe: "<tetik>", "<tetik>". For <adjacent topic>, see <other-skill>.
license: MIT
metadata:
  version: 1.0.0
  pack: goat-pen
---
```

`name` must equal the directory name: lowercase letters, digits, single hyphens.
`description` is capped at 1024 characters and must carry a `Türkçe:` segment with at
least two quoted Turkish phrases a user would really type. The skill body stays in
English; the trigger fires on Turkish input.

## Body shape

In this order:

1. Title and a one paragraph statement of the role and the outcome.
2. **Before you start** — context to read, questions to ask only if unanswered.
3. The method, 3 to 7 H2 sections. Decision tables over prose. Bad example next to good.
4. **Output format** — exactly what gets produced.
5. **Definition of done** — a checklist where every item is verifiable by reading the output.
6. **Related skills** — cross-pack links written as `pack/skill`.

## House style

- Second person for the agent: "You are reviewing the pricing page."
- Concrete over abstract. A number, a file path, or a real example beats an adjective.
- Real data only. Cite the source and the date. An invented benchmark is worse than none.
- No em dash as a sentence separator. Write the plain sentence.
- Turkish characters are written properly: ş, ç, ö, ü, ğ, ı.
- Keep `SKILL.md` under 500 lines. Detail belongs in `references/`.

## Versioning

Bump `metadata.version` in the skill on any shipped change and mirror it in
`VERSIONS.md`. Users compare that table against their installed copy, so an unbumped
change is invisible to them.

Pack version in both manifests moves in the same commit: major for layout changes,
minor for a new skill, patch for changes to existing skills.

## Skill names are global

Skills install into a flat `.claude/skills/` directory in the user's project. Two packs
shipping the same skill name means one silently overwrites the other. Before adding a
skill, check the name against the registry in the `goat-packs` working repo.

## Safety

- `install.sh` writes only inside the target project directory the user names. No home
  directory writes, no `sudo`, no global installs.
- Scripts under `skills/*/scripts/` are zero dependency and must run read-only unless
  the skill states otherwise.
- Never copy text from another skill pack. Take the idea, write it in our voice.

---
> Source: [goatstarter/goat-pen](https://github.com/goatstarter/goat-pen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
