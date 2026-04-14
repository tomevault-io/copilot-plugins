## agent-blueprint

> Follows `AGENT_BLUEPRINT.md` (version: 2026-03-16)

# AGENTS

Follows `AGENT_BLUEPRINT.md` (version: 2026-03-16)

## Project Overview

This repository defines and maintains a portable agent operating standard. The primary deliverables are the blueprint (`AGENT_BLUEPRINT.md`) and optional UI companion (`DESIGN_SYSTEM_GUIDE.md`), plus a small audit utility script.

## Stack

- Markdown documentation
- Bash scripting (`collect-project-docs.sh`)
- Git-based workflow
- No runtime application/database in this repo

## Environment

- Version manager: n/a (no runtime application)
- Version file: n/a
- Lockfile: n/a
- Setup: n/a

## Commit Trailer Template

```text
Co-authored-by: [AI_PRODUCT_NAME] <[AI_PRODUCT_EMAIL]>
AI-Provider: [AI_PROVIDER]
AI-Product: [AI_PRODUCT_LINE]
AI-Model: [AI_MODEL]
```

## Autonomous Commit Identity

- GitHub Actions roadmap workflows in this repo should use stage-specific project-relative identities:
  - `Agent Blueprint Implementer <agent-blueprint-implementer@users.noreply.github.com>`
  - `Agent Blueprint Reviewer <agent-blueprint-reviewer@users.noreply.github.com>`
  - `Agent Blueprint Fixer <agent-blueprint-fixer@users.noreply.github.com>`
- Model attribution still belongs in `Co-authored-by` plus the `AI-*` trailers; the workflow-owned git author/committer should stay separate from model attribution.
- In downstream repos, rename these identities relative to the adopting project and optionally swap the email domain to one you control.

Template rules:
- `AI_PRODUCT_LINE`: `codex|claude|gemini|opencode`
- Product-line derivation:
  - Codex or ChatGPT coding agent -> `codex`
  - Claude Code -> `claude`
  - Gemini CLI -> `gemini`
  - OpenCode -> `opencode` (regardless of underlying provider/model, including z.ai)
- `AI_PROVIDER` and `AI_MODEL`: runtime-derived at commit time
- `AI_PRODUCT_NAME` and `AI_PRODUCT_EMAIL`: resolved from **model name** using tiered lookup (see `AGENT_BLUEPRINT.md` `[BP-WF-COMMIT]`):
  1. **Brand match** (case-insensitive against model name):
     - `codex` → `Codex <codex@users.noreply.github.com>`
     - `claude` → `Claude <claude@users.noreply.github.com>`
     - `gemini` → `Gemini <google-gemini@users.noreply.github.com>`
     - `glm` → `GLM <zai-org@users.noreply.github.com>`
  2. **Provider fallback** (no brand match — use provider's GitHub org):
     - OpenAI → `OpenAI <openai@users.noreply.github.com>`
     - Anthropic → `Anthropic <anthropics@users.noreply.github.com>`
     - Google → `Google <google-gemini@users.noreply.github.com>`
     - Zhipu → `Zhipu <zai-org@users.noreply.github.com>`
     - Mistral → `Mistral <mistralai@users.noreply.github.com>`
     - Meta → `Meta <meta-llama@users.noreply.github.com>`
     - DeepSeek → `DeepSeek <deepseek-ai@users.noreply.github.com>`
  3. **Unknown**: `{Provider Name} <{github-org}@users.noreply.github.com>` or `AI Agent <noreply@users.noreply.github.com>`
- Do not store filled runtime values in this file
- For multi-model commits, see `AGENT_BLUEPRINT.md` `[BP-WF-COMMIT-MULTI]` — add one `Co-authored-by` line per contributing model and comma-separate the other trailers

## Validation Commands

| Level | Command | When |
|-------|---------|------|
| 1 | `bash -n collect-project-docs.sh` | After script changes |
| 2 | `rg -n "^version:" AGENT_BLUEPRINT.md` | After blueprint edits |
| 3 | `rg -n "BP-CORE-01|BP-ALIGN-REPORT|BP-RM-DOR" AGENT_BLUEPRINT.md` | Before completing blueprint changes |
| 4 | `echo "N/A: no UI/e2e in this repository"` | Always |

## Execution Modes

Use one policy file for both paired local work and autonomous GitHub Actions runs. Shared repo rules always apply; runtime-specific rules override only where they differ.

### Shared Rules

- Canonical planning surface is `roadmap/`; treat the referenced work unit as the source of scope.
- Roadmap work unit filenames use 3-digit IDs in this repo: `[ID]-[slug].md`.
- Use the validation commands above when their trigger conditions apply.
- Prefer a staged validation model: fast agent-run checks before PR update, then repository PR workflows for the heavier validation suite.
- Keep changes minimal and focused on the requested work unit.
- `git status` and `git diff` are always allowed for change review.
- `rg`, `sed`, `cat`, `nl`, and `wc` are always allowed for inspecting docs and scripts.
- `bash -n collect-project-docs.sh` is allowed after script changes.

### Runtime: Interactive Local

- Default mode when working directly with the user in a local agent session.
- Require user confirmation before `git commit`.
- Require user confirmation before dependency install or upgrade.
- Require user confirmation before network calls or other external side effects.
- Ask before destructive actions or anything not clearly covered by the allowlist.
- It is acceptable to stop for clarification when scope is ambiguous.

### Runtime: Autonomous GitHub Actions

- Applies to workflow-driven agent runs such as roadmap execution in GitHub Actions.
- The workflow input, especially `roadmap_path`, identifies the work unit; the referenced roadmap file is the canonical brief.
- `implement` runs should execute the fast validation commands that fit inside the agent runtime before updating a branch or PR.
- Heavier validation such as full integration or e2e suites should run in separate PR workflows after the implementation PR is updated.
- If a PR is created by `GITHUB_TOKEN`, downstream PR validation may need explicit workflow dispatch because GitHub does not recursively trigger every event from workflow-authored activity.
- `git commit`, branch creation, push, and PR creation are allowed when required to complete the referenced roadmap work unit.
- `review` runs should target an explicit PR, publish a GitHub PR review artifact, and avoid mutating code.
- `fix` runs should target an explicit existing PR, inspect review/check context first, and update that same PR branch rather than opening a replacement PR.
- For the current POC, looping between implement, review, and fix may be manually triggered between workflow runs; do not assume an unbounded autonomous loop.
- Network access is allowed when required for task execution, including GitHub operations, model-provider calls, package downloads, and task-scoped documentation lookup.
- Use GitHub Actions secrets and committed repo config; do not depend on local machine state.
- Do not pause for human confirmation steps that cannot occur inside the workflow.
- Do not browse or perform open-ended research unless the work unit requires it.
- If blocked by a true ambiguity or missing prerequisite, fail clearly in logs rather than inventing scope.

## Autonomous Review Rubric

- Review the PR against the referenced roadmap work unit first, then against general code quality.
- Blocking findings should be limited to correctness, validation gaps, security/safety issues, or clear roadmap mismatches.
- Non-blocking suggestions should stay concise and should not block approval on style-only preferences.
- Approval is appropriate only when the implementation matches the roadmap intent and the relevant PR validation checks are green or explicitly accounted for.
- Request changes when there is at least one blocking finding or when required validation is failing without an accepted explanation.
- If the state is informative but not ready for approval or blocking, leave a comment review instead of forcing approval semantics.

## Never Run

- `git reset --hard` — destructive history rewrite
- `rm -rf` — destructive deletion
- `matrix-reloaded` — disallowed in agent sessions; use provided instructions/schema instead
- Edits outside repo root — out of scope

## Project-Specific Rules

- Keep `README.md` human-facing and adoption-focused.
- Keep `AGENT_BLUEPRINT.md` as the canonical operational spec.
- Keep `CLAUDE.md` and `GEMINI.md` as thin pointers to this file.
- Preserve one-file portability of the blueprint across projects.
- Keep language concise and deterministic; avoid unnecessary ceremony.
- For workflow-owned screenshots, use `<project-root>/docs/evidence/<work-unit-slug>/` as the canonical artifact directory.
- Treat the PR body `Evidence` section as the canonical screenshot surface; use PR comments only for supplemental timeline context such as refreshed post-fix evidence.
- Workflow-authored screenshot comments and review comments should include visible provider/product/model attribution plus a machine-checkable marker.
- Demo or proof PRs are artifacts first: close superseded demos with a final pointer, and do not merge a demo PR unless it is intentionally the long-lived change vehicle.

## Codebase Reconnaissance

Run these git commands when applying the blueprint to an existing codebase or auditing a mature blueprint-following project. The output informs AGENTS.md sections like project-specific rules, validation priorities, and key files — grounding them in observed risk rather than assumptions.

| Signal | Command | What it reveals |
|--------|---------|-----------------|
| Churn hotspots | `git log --format=format: --name-only --since="1 year ago" \| sort \| uniq -c \| sort -nr \| head -20` | Files that change most often — candidates for tighter validation or ownership rules |
| Bus factor | `git shortlog -sn --no-merges --since="6 months ago"` | Knowledge concentration — flag areas where a single contributor owns 60%+ of recent changes |
| Bug clusters | `git log -i -E --grep="fix\|bug\|broken" --name-only --format='' \| sort \| uniq -c \| sort -nr \| head -20` | Files with the most bug-related commits — cross-reference with churn to find highest-risk code |
| Project momentum | `git log --format='%ad' --date=format:'%Y-%m' \| sort \| uniq -c` | Commit frequency by month — reveals team health, departures, or batch-release patterns |
| Firefighting frequency | `git log --oneline --since="1 year ago" \| grep -iE 'revert\|hotfix\|emergency\|rollback'` | Revert/hotfix rate — frequent entries suggest deploy process or test coverage gaps |

Use the results to:
- Prioritize which areas need validation commands or stricter review.
- Identify files that warrant explicit ownership or focused test coverage.
- Calibrate project-specific rules to actual risk patterns rather than convention alone.

Source: [The Git Commands I Run Before Reading Any Code](https://piechowski.io/post/git-commands-before-reading-code/) — Ally Piechowski

## Reality Check

Before implementing, briefly consider:

- [ ] **Simpler path?** — Is complexity justified by value?
- [ ] **Existing solution?** — Am I reinventing or missing a library/pattern?
- [ ] **Right problem?** — Is the framing sound, or am I treating symptoms?
- [ ] **Known pitfalls?** — Are there anti-patterns or failure modes to avoid?

If any answer gives pause, flag it before proceeding.

## Decision Artifacts

- For high-impact or irreversible decisions, create `.decisions/[name].json`.
- Use `matrix-reloaded` format for structured comparison.
- Do not run `matrix-reloaded` CLI commands; use project-provided instructions/schema only.
- Optional: add `.decisions/[name].md` for human-readable narrative context.
- Treat JSON decision matrices as the authoritative record.

## References

- For operating rules, see `AGENT_BLUEPRINT.md`
- For active work units and execution prompts, see `roadmap/`
- For decision artifacts and matrix format, see `AGENT_BLUEPRINT.md` section `Decision Artifacts [BP-DECISIONS]`
- For UI system workflows, see `DESIGN_SYSTEM_GUIDE.md`
- For cross-project audit collection, see `collect-project-docs.sh`

## Key Files

- `AGENT_BLUEPRINT.md` — canonical blueprint
- `AGENTS.md` — repo-specific policy entrypoint
- `README.md` — human adoption guide
- `DESIGN_SYSTEM_GUIDE.md` — optional UI design system guide
- `collect-project-docs.sh` — multi-project reference collector
- `roadmap/index.md` — project roadmap and work unit directory
- `.decisions/` — decision artifacts (matrix-reloaded JSON records)

## Knowledge Base

Tool: Roam Research

When asked to generate a Roam summary or thread:
- Parent block: `- [[<tool>]] [[<model-id>]] [[ai-thread]] [[agent-blueprint]]`
- Tool names: `opencode` | `claude-code` | `gemini-cli` | `codex-cli`
- Page refs: only include `[[Page Name]]` if explicitly instructed
- Sections: ask user what they want (chronological, functional, Q&A)

## User Profile

See `.agent-profile.md` (git-ignored) for interaction preferences.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jgoodhcg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
