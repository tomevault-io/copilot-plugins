## software-architecture-claude-plugin

> Orientation for agents working in this repo. For the full feature list see

# AGENTS.md — Software Architecture plugin

Orientation for agents working in this repo. For the full feature list see
[README.md](README.md); for how to add or change skills/commands/agents/hooks
see [CONTRIBUTING.md](CONTRIBUTING.md). This file covers what those don't:
where things live, the knowledge stores, and the working conventions.

## What this repo is

The `software-architecture` Claude Code plugin — cross-product architecture
practice (ADRs, fitness functions, technology radar, multi-level governance,
Hard Parts trade-off analysis) delivered as skills, commands, agents, and hooks.
Grounded in Neal Ford's *Fundamentals of Software Architecture* and *The Hard
Parts*. Scope is **cross-product**; product-specific content lives in that
product's own plugin (see CONTRIBUTING.md for the dividing line).

## Repo map

```
skills/        # 10 architecture-practice skills (the product)
commands/      # /software-architecture:<verb> commands
agents/        # adr-historian, fitness-author, architecture-reviewer
hooks/         # 5 hooks (opt-out via SA_PLUGIN_HOOKS)
docs/adr/      # ADRs about THIS plugin's own structure (dogfooding)
docs/solutions/                  # documented solutions to past problems (see below)
# docs/brainstorms/, docs/plans/ — created on demand by ce-brainstorm / ce-plan
CONCEPTS.md                      # shared domain vocabulary (see below)
```

## Knowledge stores

These accumulate institutional knowledge — consult them when orienting or
before working in an area they cover.

- **`docs/solutions/`** — documented solutions to past problems (bugs, best
  practices, architecture patterns, workflow learnings), organized by category
  with YAML frontmatter (`module`, `tags`, `problem_type`, `component`).
  Relevant when implementing or debugging in a documented area. Written by
  `ce-compound`; maintained by `ce-compound-refresh`.
- **`CONCEPTS.md`** — shared domain vocabulary (architectural characteristic,
  fitness function, decision archetype, governance level, ADR, technology radar,
  and the cross-plugin-skill terms). Relevant when orienting to the codebase or
  discussing domain concepts.
- **`docs/adr/`** — decisions about the plugin itself. Read the relevant ADR
  before changing structure it governs; capture new structural decisions as ADRs
  (`adr new <title>`).

## Working conventions

Full detail in [CONTRIBUTING.md](CONTRIBUTING.md). The essentials:

- **Branches:** `claude/<slug>` for Claude-authored work (enables auto-merge on
  green CI), `adr/<slug>` for plugin-local ADR work, `feat|fix|chore/<slug>` for
  human work.
- **Commits:** Conventional Commits (`feat(scope):`, `fix:`, `chore:`, `docs:`,
  `ci:`). Never bypass CI (`--no-verify` is out of bounds).
- **Decisions:** structural/architectural decisions about the plugin land as ADRs
  in `docs/adr/`. The plugin eats its own dogfood.
- **Cross-references** between skills use the full namespace
  (`software-architecture:<skill-name>`), never the bare name.
- **Testing:** `npm test` runs the `node:test` suite in `test/` (no
  third-party deps). It checks frontmatter validity (skills, commands,
  agents), that hook scripts are executable and emit valid JSON, and that
  the `.claude-plugin/` manifests are valid JSON. See CONTRIBUTING.md for
  the per-file breakdown.

## Distribution

Published from this repository. Consumers add it as a Claude Code plugin
marketplace and install the `software-architecture` plugin (see README).
Merging to `main` is the release surface — keep `main` releasable.

---
> Source: [Integral-Productivity/software-architecture-claude-plugin](https://github.com/Integral-Productivity/software-architecture-claude-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
