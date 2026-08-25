## agent-ci

> This repo is a bootstrapper for the agent-ci coordination system. It contains the canonical versions of six coordination skills and the `init` flow that installs them into a target repo. See [README.md](README.md) for the full pitch.

# Agent-CI Kit

This repo is a bootstrapper for the agent-ci coordination system. It contains the canonical versions of six coordination skills and the `init` flow that installs them into a target repo. See [README.md](README.md) for the full pitch.

## What this repo is

- `.claude/skills/` — the six coordination skills, including `init`
- `templates/` — files that `init` copies and customizes into target repos:
  - `CLAUDE.md` — the filled-in-style root template (init substitutes placeholders against this)
  - `CLAUDE-TEMPLATE.md` — the guidance-annotated reference (init ships this verbatim to target's `docs/`)
  - `coordination/schema.yaml` — the manifest schema
  - `worktreeinclude` — copied to the target's `.worktreeinclude` (gitignored env files to copy into parallel-session worktrees)
  - `source-of-truth/` — six stub templates for the per-domain canonical docs (`README`, `ARCHITECTURE`, `DATA-MODEL`, `API-REFERENCE`, `MODULE-TEMPLATE`)
  - `github/` — PR template, issue templates, CODEOWNERS skeleton, and `workflows/` (the agent pipeline + an opt-in correctness CI)
- `README.md` — the methodology, the quick start, the rationale for what's intentionally absent

This is **not** a typical application repo. The coordination skills here (`coordination-check`, `start-work`, etc.) are for use in target repos, not for coordinating work on this kit itself.

`init` produces four things in a target repo: (1) a tailored CLAUDE.md, (2) a scaffolded `docs/source-of-truth/` with per-domain stubs, (3) the coordination system + GitHub plumbing, (4) the autonomous agent pipeline (triage → auto-fix → plan workflows). Brownfield repos are handled by prompting for merge strategy on existing files rather than overwriting silently.

## Working on the kit

Standard git workflow: branch, commit, PR. No `.coordination/<branch>.yaml` manifests — that machinery is for target repos.

- The `init` skill IS meant to run from here, pointed at a target repo path.
- The other coordination skills travel with the target repo. Don't run them against the kit.
- When you change a skill, test it against a real target repo before merging.

## Key paths

- `.claude/skills/init/` — the entry point; reads from `templates/` and writes to a target repo
- `.claude/skills/<other>/` — operational skills that get copied to target repos
- `templates/CLAUDE.md` — the root CLAUDE.md template deployed to target repos (has placeholders)
- `templates/coordination/schema.yaml` — the manifest schema
- `templates/github/` — PR template, issue templates, CODEOWNERS skeleton
- `templates/github/workflows/` — the agent pipeline (`agent-triage.yml`, `agent-autofix.yml`, `agent-plan.yml`) + an opt-in correctness CI (`ci.yml`)

## Two layers: advisory coordination, autonomous issue handling

Keep these distinct when working on the kit:

- **Coordination is advisory** (skills only). The kit ships **no coordination-*enforcement*** workflows — no manifest validator, no auto-refresher, no collision dashboard. Two repos tried that and removed it. If you're tempted to add coordination enforcement, do it in a specific target repo for a specific failure mode — don't bake it into the kit.
- **Issue handling is autonomous** (the agent pipeline). The `agent-*.yml` workflows are core and ship with every `init`. They route issues by label to a coding agent or a planning agent. The label contract + guardrails that govern them live in `templates/CLAUDE.md` (folded into the target's CLAUDE.md), **not** in a separate agent-rules file. The auto-fix toolchain is the one stack-specific piece — `init` substitutes the fenced `AGENT_TOOLCHAIN_SETUP` block in `agent-autofix.yml`.

When changing a workflow, test it against a real target repo (it needs the Claude GitHub app + an `ANTHROPIC_API_KEY` secret) before merging.

---
> Source: [rdrakefambrands/agent-CI](https://github.com/rdrakefambrands/agent-CI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
