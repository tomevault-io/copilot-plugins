## pragmarketplace

> Guidance for Codex agents working in `pragmabits/pragmarketplace`.

# AGENTS.md

Guidance for Codex agents working in `pragmabits/pragmarketplace`.

## Project Identity

This repository is a plugin marketplace. It currently publishes Claude Code plugins under `plugins/claude/` and keeps the marketplace registry in `.claude-plugin/marketplace.json`. There is no application code, build pipeline, package manager workflow, or in-repo test runner.

Codex-specific work belongs outside the Claude plugin space. Use `AGENTS.md`, `.codex/`, and `plugins/codex/` for Codex instructions, rules, experiments, and future Codex plugin content.

## Hard Boundary: Claude Plugin Space Is Read-Only

Codex agents may read, inspect, summarize, validate, and review the Claude plugin space, but must never create, edit, move, delete, format, or mechanically rewrite anything under these paths:

- `plugins/claude/`
- `.claude-plugin/`
- `.claude/`
- `CLAUDE.md`

This includes manifests, commands, skills, agents, hooks, scripts, references, docs, output styles, marketplace metadata, and session files. If a user asks for a change to the Claude plugin space, explain that this Codex configuration treats those paths as read-only and ask them to switch to the Claude workflow or explicitly change the repository policy first.

Do not use shell redirection, `tee`, `sed -i`, ad hoc scripts, formatters, code generators, `git checkout`, `git restore`, or any other workaround to modify those paths. The local Codex filesystem permission profile in `.codex/config.toml` enforces read-only access for the same boundary.

## Repo Layout

- `.claude-plugin/marketplace.json`: central Claude plugin registry and published version source of truth.
- `plugins/claude/<plugin>/.claude-plugin/plugin.json`: per-plugin Claude manifest.
- `plugins/codex/`: reserved Codex plugin space.
- `.agents/plugins/marketplace.json`: Codex marketplace metadata for repo-local Codex plugins.
- `.sessions/`: agent-agnostic session handoff reports. This belongs to the user project, not to Claude, Codex, `.agents`, or any plugin runtime.
- `.codex/config.toml`: project-local Codex configuration and filesystem permission profile.
- `.codex/rules/*.rules`: project-local Codex exec approval rules.
- `CLAUDE.md`: Claude Code instructions. Read-only for Codex agents.

## Registered Claude Plugins

Current marketplace entries:

- `git` `3.0.1`: semantic commit workflow, Conventional Commits validation, native git hooks, safety hooks.
- `frontend` `1.8.0`: frontend orchestrator with CSS, Tailwind CSS, Vue.js, Nuxt, shadcn/ui, Font Awesome, Material Design 3, HTMX + Go, and design specialists.
- `review` `1.0.1`: `/codex-review`, backed by `codex-review.sh` and the external Codex CLI.
- `session` `2.2.0`: `/report` and `/recall` handoff reports for Claude Code.
- `pragmatic` `1.0.1`: pragmatic Claude output style.
- `architect` `0.1.2`: decision-support and scaffolding for Claude sub-agents and loops.

Treat those entries as read-only inventory unless the project policy changes.

## Validation

For Codex-side changes, keep verification structural and local:

```bash
git status --short
python3 -c 'import pathlib, tomllib; tomllib.loads(pathlib.Path(".codex/config.toml").read_text())'
codex execpolicy check --pretty --rules .codex/rules/claude-readonly.rules -- rm -rf plugins/claude/git
```

`jq` is for JSON only; do not run it on TOML.

## Working Conventions

- Prefer `rg` and `rg --files` for discovery.
- Keep Codex edits scoped to Codex-owned files unless the user explicitly changes the read-only boundary.
- Store session handoff reports under root `.sessions/`; never write session reports under `.codex/`, `.agents/`, or `.claude/`.
- Do not introduce build tooling or dependency files unless the task specifically requires it.
- Keep plugin marketplace facts synchronized with `.claude-plugin/marketplace.json` when documenting inventory, but do not edit that registry from Codex.
- When a request touches both Claude and Codex concerns, implement only the Codex-owned side and clearly name any Claude-side change that remains blocked by policy.

---
> Source: [pragmabits/pragmarketplace](https://github.com/pragmabits/pragmarketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
