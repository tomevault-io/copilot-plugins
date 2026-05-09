## mint

> Disciplined agentic development framework for Claude Code.

# mint

Disciplined agentic development framework for Claude Code.

<!-- mint:start v2 -->
## MANDATORY: Use mint for ALL Code Changes

**For ANY task that modifies files in this repo, invoke the `mint` skill FIRST.**

This is not optional. Before writing, editing, or deleting any code:
1. Invoke `mint` with the task description
2. mint auto-routes to the right mode (quick/plan/ship/research/verify)
3. Follow mint's execution flow with gates and reviews

The only exceptions:
- Pure conversation / answering questions
- Reading files to understand context (no modifications)

**NEVER use Claude Code's built-in plan mode (EnterPlanMode/ExitPlanMode).** mint has its own planning flow.
<!-- mint:end -->

## Commands

- Test: `bun test`
- Lint/Types: checked by mint gates via `.mint/config.json`
- Version bump: `./scripts/bump.sh [major|minor|patch]` — updates all version locations, does not commit

## Code Style

- No AI attribution — never add `Co-Authored-By` or mention AI tools in commits
- Commits: `type(scope): description` — see `docs/conventions.md:66-72` for types
- Branches: `feat/<name>` or `fix/<name>` off main. Squash merge via PR
- Never push from agents — commit only, human reviews and pushes

## Architecture

- `skills/mint/SKILL.md` — thin router (~125 lines), loads mode/phase files on demand
- `skills/mint/modes/` — one file per execution mode (quick, plan, ship, etc.)
- `skills/mint/phases/` — one file per pipeline step (implement, review, docs, etc.)
- `skills/mint/reference/` — detailed docs loaded only when needed
- `.mint/skills/` — auto-generated project-specific skills from adaptive automation
- `agents/*.md` — one agent per job, max 200 lines each
- CLI uses bun + @clack/prompts. See `cli/` and `cli/lib/`
- Reviewers use three severities: BLOCKING, WARNING, INFO
- See `standards/agent-prompts.md` for how to write agent prompts
- See `standards/claude-md.md` for how to write CLAUDE.md files

## Key Docs

- `docs/architecture.md` — system design, philosophy, isolation rules
- `docs/conventions.md` — file formats, naming, config schema, git strategy
- `templates/spec.xml` — XML spec schema every task gets
- `.mint/config.json` — project config (gates, reviewers, browser, design, plugins)
- `.mint/hard-blocks.md` — immutable constraints agents can never violate
- `.mint/research/adaptive-automation.md` — adaptive automation design

## Gotchas

- Design plans go in `docs/plans/` (gitignored) — don't commit them
- `.mint/sessions/` files are gitignored — per-session state, not shared
- All learning logs (issues, wins, instincts) are JSONL with confidence scoring — use `upsertInstinct()`, never raw append
- The old 1900-line SKILL.md is gone — router + modes + phases + references
- `.mint/skills/` is gitignored — generated skills are personal, promote to `.claude/skills/` to share
- When user mentions automating a workflow, check `.mint/skills/` for generated skills and `.mint/workflow-candidates.jsonl` for detected patterns
- No superpowers plugin — mint is the orchestration framework, don't layer another on top

---
> Source: [3li7alaki/mint](https://github.com/3li7alaki/mint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
