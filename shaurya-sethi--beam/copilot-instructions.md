## beam

> This repository contains **Beam**, a [superpowers][sp]-style skills plugin

# Repository Guidelines

## Project Purpose & High-Level Overview

This repository contains **Beam**, a [superpowers][sp]-style skills plugin
for agent-driven harness engineering. It is a directory of cooperating
`SKILL.md` files (plus a few template assets) that coding agents load to
guide a four-phase repository initialization workflow:

1. Goal elicitation (Q&A → `plan.md` + session todos)
2. Repo scaffolding (`collab_progress/` + minimal starter structure)
3. `AGENTS.md` generation from `assets/agents_config.json`
4. Technical spec (web research + `docs/SPEC.md`)

There is no compiled code in this project. The entire deliverable is
Markdown and JSON. Primary audiences are maintainers improving the skills,
and the coding agents that load and execute them.

[sp]: https://github.com/obra/superpowers

## Project Structure & Module Organization

```text
.
├── AGENTS.md
├── README.md
├── LICENSE
├── .markdownlint.json
├── collab_progress/                 # Progress notes, CHANGELOG, PROTOCOL
├── docs/
│   └── SPEC.md
└── skills/
    ├── using-beam/
    │   └── SKILL.md                    # Master bootstrap skill;
    │                                   # triggers on new-project intent
    ├── eliciting-project-goals/
    │   └── SKILL.md                    # Phase 1: Q&A → plan.md + todos
    ├── scaffolding-repo/
    │   ├── SKILL.md                    # Phase 2: collab_progress +
    │   │                               # starter structure + README.md
    │   ├── readme-template.json        # README.md section structure
    │   └── collab_progress_config.json # collab_progress/ file contents
    ├── writing-agents-md/
    │   ├── SKILL.md                    # Phase 3: walks agents_config.json
    │   │                               # → AGENTS.md
    │   └── agents_config.json          # AGENTS.md section structure
    └── writing-technical-spec/
        ├── SKILL.md                    # Phase 4: web research + SPEC.md
        └── spec-template.json          # SPEC.md section structure
```

## Build, Test, and Development Commands

There is no build step. The "commands" for working on Beam are the
install, baseline-test, and verify-change loop that the
`superpowers:writing-skills` skill describes in detail.

| Operation | How |
| --------- | --- |
| Install locally for testing | Symlink `./skills` into your agent's skill dir, e.g. `ln -s "$PWD/skills" ~/.agents/skills/beam` |
| Run a baseline (RED) | Open a fresh agent session **without** the changed skill loaded, and ask it to perform the workflow; record where it fails or rationalizes |
| Verify a change (GREEN) | Re-run the same baseline scenario **with** the updated skill; confirm the agent now complies |
| Lint the Markdown | Use any `markdownlint`-compatible tool against the repo root; rules live in `.markdownlint.json` |

The full discipline — RED → GREEN → REFACTOR for skills — is documented in
the `superpowers:writing-skills` skill. Read it before editing any
`SKILL.md` in this repo.

## Coding Style & Naming Conventions

This repo is Markdown and JSON only. Style rules are about file layout and
SKILL frontmatter, not code.

- **Markdown line length:** 80 characters (tables exempt). Enforced by
  `.markdownlint.json`.
- **SKILL.md frontmatter:** YAML with two required fields:
  - `name` — letters, numbers, hyphens only; ≤64 characters.
  - `description` — third-person, starts with `"Use when..."`, ≤1024
    characters. Describes **triggers only** — never summarize the
    workflow inside the skill body, or agents will follow the description
    instead of reading the skill.
- **Skill directory names:** kebab-case, gerund-first
  (`eliciting-project-goals`, `writing-technical-spec`).
- **Template / reference files inside a skill:** lowercase with hyphens
  (`readme-template.json`, `spec-template.json`).
- **JSON templates alongside their skill:** each template
  (`readme-template.json`, `collab_progress_config.json`,
  `agents_config.json`, `spec-template.json`) lives in its owning skill's
  directory. Schemas are stable; document any field changes in
  `collab_progress/`.

## Testing Guidelines

Skills are tested using the TDD-for-skills process from the
`superpowers:writing-skills` skill. There is no automated test runner.

For every change to a `SKILL.md` or template that affects agent behavior:

1. **RED — Baseline.** Run the target scenario in a fresh subagent
   session **without** your change. Document the agent's behavior verbatim
   — every rationalization, every skipped step.
2. **GREEN — Write the change.** Make the smallest edit that addresses
   the observed failures. Re-run the same scenario; confirm the agent
   now complies.
3. **REFACTOR — Close loopholes.** If the agent finds a new
   rationalization, add it to a rationalization table in the skill, then
   re-test until the behavior is bulletproof.

Discipline-enforcing skills (anything with a `<HARD-GATE>` or
"do not proceed until..." rule) **must** include a rationalization table
once they have been pressure-tested.

## Commit & Pull Request Guidelines

- **Commits:** imperative, present-tense subject (e.g. `"Add phase-2
  scaffolding skill"`), ~72 characters. Add a body for context when the
  change affects agent behavior or skill discovery.
- **Pull requests:** short summary, link related issues or design notes,
  call out any breaking change to skill triggers or template schemas.
- **Behavior-affecting changes:** any edit to a `SKILL.md` or to a file
  in `assets/` that changes what an agent will do **must** be accompanied
  by a brief verification note in `collab_progress/` (see the protocol
  section below) describing the baseline, the change, and the verified
  outcome.

## Documentation & Knowledge Sources

ALWAYS reference official documentation for libraries/frameworks before
implementing anything new or changing existing behavior.
> Harness Engineering reference: <https://openai.com/index/harness-engineering/>

### **Primary Sources**

1. **[docs/SPEC.md](docs/SPEC.md)** for project scope, architecture, and
   non-goals before designing or modifying skill behavior.
2. **[agentskills.io/specification](https://agentskills.io/specification)**
   for canonical SKILL.md authoring rules (frontmatter fields, length
   limits, discovery semantics).
3. Official library and framework documentation on the internet. Use your
   built-in `web search` tools.
4. If all else fails, browse dependency source locally (for example
   through your package manager cache, a vendored directory, or published
   source tarballs and repository tags). Ask the user to add or update
   dependencies if needed so the relevant code is available to you.

## Collaboration & Progress Tracking Protocol

**Before starting any new task:**

- The `collab_progress` folder exists to track and understand the current state,
recent changes, and overall repo progress.
- Decide whether reviewing these is necessary for the given task. Then `ls` the
folder contents to view file names and check relevant files if you so decide.

**After completing any significant change:**

- Follow the instructions in `collab_progress/PROTOCOL.md` to document your work.

**This protocol is mandatory for all contributors and must be followed for every
codebase change.**

---
> Source: [Shaurya-Sethi/beam](https://github.com/Shaurya-Sethi/beam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
