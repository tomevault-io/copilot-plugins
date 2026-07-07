## ml-research-skills

> Guidance for AI coding agents working in this repository.

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Repository Overview

This repo is **ml-research-skills** — a collection of agent skills for ML researchers. Each skill lives under `skills/<skill-name>/` as an instruction bundle centered on `SKILL.md`, with optional `references/`, `scripts/`, `templates/`, and `agents/` directories.

This repo is **one pack** in the broader portable [Skill OS matrix](https://github.com/a-green-hand-jack/skill-os) for AI research workflows. The matrix has six sibling pack repos plus a hub repo: [`skill-os`](https://github.com/a-green-hand-jack/skill-os) (hub: kernel schema, install/repo-split contract, installer, registry), [`core-ops-skills`](https://github.com/a-green-hand-jack/core-ops-skills) (substrate), [`automation-skills`](https://github.com/a-green-hand-jack/automation-skills), [`paper-reading-skills`](https://github.com/a-green-hand-jack/paper-reading-skills), [`research-distillation-skills`](https://github.com/a-green-hand-jack/research-distillation-skills), [`quick-experiment-skills`](https://github.com/a-green-hand-jack/quick-experiment-skills), and `ml-research-skills` (this repo, full ML research lifecycle). Kernel schema and contracts are authoritative in `skill-os`; this pack owns its own kernel example (`schemas/skill-kernel/examples/ml-research.kernel.json`) and a sliced `profiles/profile-index.yaml`. Codex / Claude / future agents are adapter targets, not the workflow source of truth.

The repo also has shared project memory under `memory/`. **Before any substantial maintenance work, read `memory/BRIEFING.md`** (the ≤30-line compact snapshot) and `memory/current-status.md`. Update the smallest relevant board after durable workflow, routing, validation, or architecture decisions. Keep local sidecar artifacts under `.agent/sidecars/`; they are ignored and should not be committed unless explicitly sanitized and requested.

Visual documentation assets live under `asset/`. Before adding, replacing, or renaming a figure, read `asset/README.md`; keep filenames semantic, update README links, and refresh the figure inventory in `memory/evidence-board.md` plus `memory/current-status.md` when the visual system changes materially.

Recommended global install uses only the bootstrap skill:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s ml-research-bootstrap -y
```

If the global agent must initialize new ML research project roots, also install:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s project-init -y
```

Install the full collection only from an initialized ML research project root:

```bash
npx skills add a-green-hand-jack/ml-research-skills
```

To install one specific skill globally for both agents when explicitly needed:

```bash
npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s remote-project-control -y
```

Full global installs are maintainer/debug-only because this full ML research lifecycle bundle can interfere with ordinary non-ML projects.

These files are primarily agent instructions and templates, not an application with automated runtime tests.

## Git Closeout Policy

For Git commits, pushes, branch operations, worktrees, merge/rebase/cherry-pick, lock-file errors, or sandbox/network Git failures, use `safe-git-ops`.

For routine branch pushes after preflight has identified the repo, remote, and branch, use:

```bash
project-push <repo> <remote> <branch>
```

Do not use raw `git push`, `git -C <repo> push`, `cd <repo> && git push`, or shell-wrapped push variants for routine closeout unless `project-push` is unavailable. `git -C <repo> ...` is still fine for inspection and non-push repo-local commands.

## Current Repository Structure

```text
ml-research-skills/
├── README.md
├── CLAUDE.md
├── AGENTS.md
├── asset/
│   ├── README.md
│   └── *.png
├── memory/
│   ├── project.yaml
│   ├── current-status.md
│   ├── decision-log.md
│   ├── action-board.md
│   ├── risk-board.md
│   ├── phase-dashboard.md
│   ├── claim-board.md
│   ├── evidence-board.md
│   ├── provenance-board.md
│   ├── handoff-board.md
│   ├── component-index.yaml
│   └── source-visibility-board.md
├── profiles/
│   ├── README.md
│   └── profile-index.yaml         (sliced to ml-research entry only)
├── schemas/
│   └── skill-kernel/
│       ├── README.md
│       ├── skill-kernel.schema.json
│       └── examples/
└── skills/
    ├── add-git-tag/
    │   └── SKILL.md
    ├── research-project-memory/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── research-idea-validator/
    │   ├── SKILL.md
    │   └── references/
    ├── literature-review-sprint/
    │   ├── SKILL.md
    │   └── references/
    ├── reference-library-manager/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   ├── scripts/
    │   └── templates/
    ├── reference-reading-summarizer/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── templates/
    ├── reference-project-synthesizer/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── templates/
    ├── reference-corpus-analyzer/
    │   ├── SKILL.md
    │   └── templates/
    ├── algorithm-design-planner/
    │   ├── SKILL.md
    │   └── references/
    ├── init-latex-project/
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── templates/
    ├── latex-layout-issue-bundler/
    │   ├── SKILL.md
    │   └── scripts/
    ├── init-python-project/
    │   └── SKILL.md
    ├── new-workspace/
    │   └── SKILL.md
    ├── data-pipeline-manager/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── experiment-debugger/
    │   ├── SKILL.md
    │   └── references/
    ├── compute-budget-planner/
    │   ├── SKILL.md
    │   └── references/
    ├── feedback-synthesizer/
    │   ├── SKILL.md
    │   └── templates/
    ├── appendix-organizer/
    │   ├── SKILL.md
    │   └── references/
    ├── experiment-design-planner/
    │   ├── SKILL.md
    │   └── references/
    ├── baseline-selection-audit/
    │   ├── SKILL.md
    │   └── references/
    ├── statistical-analysis-planner/
    │   ├── SKILL.md
    │   └── references/
    ├── result-diagnosis/
    │   ├── SKILL.md
    │   └── references/
    ├── research-results-auditor/
    │   ├── SKILL.md
    │   └── references/
    ├── project-pivot-planner/
    │   └── SKILL.md
    ├── experiment-report-writer/
    │   ├── SKILL.md
    │   └── templates/
    ├── advisor-update-writer/
    │   ├── SKILL.md
    │   └── references/
    ├── research-slide-deck-builder/
    │   └── SKILL.md
    ├── figure-results-review/
    │   ├── SKILL.md
    │   └── references/
    ├── table-results-review/
    │   └── SKILL.md
    ├── paper-evidence-board/
    │   ├── SKILL.md
    │   └── references/
    ├── paper-evidence-gap-miner/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-result-asset-builder/
    │   ├── SKILL.md
    │   ├── references/
    │   ├── scripts/
    │   └── templates/
    ├── paper-writing-memory-manager/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-positioning-planner/
    │   ├── SKILL.md
    │   └── references/
    ├── conference-writing-adapter/
    │   ├── SKILL.md
    │   └── references/
    ├── abstract-title-contribution-writer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── experiment-story-writer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── limitations-scope-writer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── method-section-explainer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-introduction-argument-writer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-draft-consistency-editor/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── auto-paper-improvement-loop/
    │   ├── SKILL.md
    │   └── templates/
    ├── paper-writing-contract-planner/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-writing-assistant/
    │   ├── SKILL.md
    │   └── references/
    ├── related-work-positioning-writer/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── paper-reviewer-simulator/
    │   ├── SKILL.md
    │   └── references/
    ├── rebuttal-strategist/
    │   ├── SKILL.md
    │   └── references/
    ├── camera-ready-finalizer/
    │   ├── SKILL.md
    │   └── references/
    ├── artifact-evaluation-prep/
    │   ├── SKILL.md
    │   └── references/
    ├── citation-coverage-audit/
    │   ├── SKILL.md
    │   └── references/
    ├── citation-audit/
    │   ├── SKILL.md
    │   ├── references/
    │   └── scripts/
    ├── work-timeline-planner/
    │   ├── SKILL.md
    │   ├── references/
    │   └── templates/
    ├── token-usage-auditor/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── scripts/
    ├── sidecar-task-runner/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── templates/
    │   └── scripts/
    ├── personalization-memory/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   └── templates/
    ├── code-reviewer/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   ├── scripts/
    │   └── templates/
    ├── ml-research-bootstrap/
    │   ├── SKILL.md
    │   └── agents/
    ├── project-init/
    │   └── SKILL.md
    ├── project-sync/
    │   └── SKILL.md
    ├── safe-git-ops/
    │   ├── SKILL.md
    │   ├── scripts/
    │   └── references/
    ├── remote-project-control/
    │   ├── SKILL.md
    │   ├── references/
    │   ├── scripts/
    │   ├── template_manifest.json
    │   └── templates/
    ├── model-card-writer/
    │   ├── SKILL.md
    │   └── templates/
    ├── release-code/
    │   ├── SKILL.md
    │   ├── checklist.md
    │   └── templates/
    ├── skill-system-auditor/
    │   ├── SKILL.md
    │   └── references/
    ├── run-experiment/
    │   ├── SKILL.md
    │   ├── environments.yaml
    │   └── templates/
    ├── run-status-monitor/
    │   ├── SKILL.md
    │   ├── agents/
    │   ├── references/
    │   ├── scripts/
    │   └── templates/
    ├── submit-paper/
    │   ├── SKILL.md
    │   └── scripts/
    └── update-docs/
        └── SKILL.md
```

## Current Skill Set

| Skill | Purpose |
|---|---|
| `research-idea-validator` | Turn a rough research idea into a pursue/revise/park/kill decision using novelty, feasibility, evidence, and reviewer-risk analysis |
| `algorithm-design-planner` | Turn a promising research idea into a concrete method design with formulation, mechanism, assumptions, ablations, and implementation handoff |
| `init-latex-project` | Initialize a LaTeX paper project with venue-specific templates, macros, and downloaded style files |
| `latex-layout-issue-bundler` | Create reproducible `.agent/layout-issues/` bundles from PDF pages, crops, source snippets, and compile logs so agents can debug LaTeX layout without manual screenshots |
| `init-python-project` | Create or enhance a production-ready Python/ML code repo using `uv`, `ruff`, `mypy`, `pytest`, and `pre-commit`, with code-side evidence docs and remote workflow memory |
| `project-init` | Create a research project control root with independent paper/code/slides repos, shared memory, optional GitHub Project board linkage, root project docs, root agent guidance, toolchain gates, and code/paper worktree policy |
| `project-sync` | Sync experiment results from the code repo into the paper's `daily_experiments.tex` |
| `ml-research-bootstrap` | Bootstrap project-local ml-research-skills from a thin global install: create or enable an ML research project, route new projects to `project-init`, and avoid full-bundle installs in generic non-ML repos |
| `ml-research-router` | Project-local router for ML research skill selection. Use inside initialized ML research projects, or while maintaining this repo, when the user may not know which skill to invoke; do not use for generic non-ML projects |
| `paper-writing-router` | Route paper writing tasks — contract, prose, section drafting, consistency editing, review simulation, rebuttal, submission — to the correct skill. Use instead of guessing between paper-writing-assistant, paper-writing-contract-planner, paper-reviewer-simulator, or citation skills |
| `experiment-design-planner` | Design hypothesis-driven experiments with baselines, ablations, metrics, controls, logging, and stop conditions before running |
| `baseline-selection-audit` | Audit whether experimental baselines are necessary, fair, current, and reviewer-proof before running or writing comparisons |
| `statistical-analysis-planner` | Plan and report statistical rigor for ML results: significance tests, effect sizes, confidence intervals, seed variance analysis, and multiple-comparison corrections |
| `research-results-auditor` | Audit completed experiment results for scientific validity before locking claims: confound checking, claim-drift detection, protocol integrity, and inferential quality |
| `project-pivot-planner` | Plan mid-project direction changes when consistent failures require scope narrowing, angle change, or kill decisions — distinct from per-experiment diagnosis |
| `feedback-synthesizer` | Turn inbound advisor, collaborator, or reviewer feedback into structured claim updates, risk entries, action items, and experiment decisions |
| `advisor-update-writer` | Write decision-oriented advisor, mentor, lab meeting, or collaborator updates from evidence, risks, options, asks, and next actions |
| `research-slide-deck-builder` | Design and write research slide decks for advisor, lab, progress, reading, proposal, and conference talks using the external `progress-slides` template |
| `figure-results-review` | Review figure assets, LaTeX figure wrappers, plots, captions, visual descriptions, paper visual style, and evolving style-memory contracts for claim support and reviewer risk |
| `table-results-review` | Review standalone `tables/*.tex` files, table captions, table descriptions, row/column semantics, numeric provenance, and experiment settings |
| `paper-evidence-board` | Maintain a paper-facing board aligning claims, evidence, figures, sections, reviewer risks, and next actions |
| `paper-evidence-gap-miner` | Mine existing CSV results, logs, reports, and assets to fill claim evidence gaps before planning new compute |
| `paper-result-asset-builder` | Build paper-facing tables, figures, wrappers, inventories, provenance records, and style-contract-aware plots from CSV experiment outputs |
| `paper-writing-memory-manager` | Maintain dynamic writing memory across nonlinear drafting sessions, section status, dependencies, writing layers, style decisions, edit impact, stale prose, and open writing threads |
| `paper-positioning-planner` | Decide the paper's primary contribution, claim scope, archetype, target audience, novelty framing, and claims to avoid before venue-specific writing |
| `conference-writing-adapter` | Adapt an ML paper's structure, positioning, and paragraph-level writing to a target conference using venue exemplars and reusable writing memory |
| `abstract-title-contribution-writer` | Draft and revise titles, abstracts, and contribution lists so the paper's top-level promise matches venue, positioning, claims, and evidence |
| `experiment-story-writer` | Turn experiment tables, figures, ablations, mixed results, and provisional metrics into claim-aware results prose |
| `limitations-scope-writer` | Plan, draft, and revise limitations, scope, failure cases, ethics, broader impact, and conclusion caveats as claim-boundary control |
| `method-section-explainer` | Plan, draft, and revise method sections for notation flow, module ordering, overview figures, algorithm boxes, design rationales, and appendix boundaries |
| `paper-introduction-argument-writer` | Plan, draft, and revise introductions as venue-aware argument chains with hook, gap, insight, method, evidence, and contribution paragraph roles |
| `paper-draft-consistency-editor` | Audit and edit a paper draft for internal consistency across title, abstract, intro, method, results, figures, tables, captions, terminology, writing layers, limitations, and conclusion |
| `auto-paper-improvement-loop` | Run multi-round review-implement-recompile cycles on a paper draft with reviewer independence (fresh context per round), edit-whitelist gating, and crash-resumable state |
| `paper-writing-contract-planner` | Create or update a writing contract that locks paper archetype, section order, paragraph roles, claim evidence slots, writing-layer permissions, figure/table jobs, and forbidden claims before drafting |
| `paper-writing-assistant` | Draft and revise claim-aware paper prose, map archetypes to required evidence slots, use micro-patterns for captions and paragraph-level writing, and track provisional result placeholders until verified evidence arrives |
| `related-work-positioning-writer` | Plan, draft, and revise related work as novelty-boundary writing, grouping closest work and defining safe citation-backed boundary statements |
| `paper-reviewer-simulator` | Simulate target-conference reviewers, predicted scores, likely reject reasons, meta-review, rebuttal risks, and a ranked pre-submission risk register |
| `rebuttal-strategist` | Analyze real reviews, infer reviewer intent, plan rebuttal experiments, draft responses, and track promised revisions |
| `appendix-organizer` | Plan and write appendix or supplementary material: structure sections, enforce main-paper claim boundaries, fill NeurIPS/ICLR/ICML reproducibility checklists, and align cross-references |
| `camera-ready-finalizer` | Finalize an accepted paper by checking rebuttal promises, de-anonymization, final claims/evidence, supplement consistency, submission package, and release handoff |
| `artifact-evaluation-prep` | Prepare artifact evaluation packages, reviewer-facing reproduction instructions, smoke tests, manifests, and claim-to-artifact maps |
| `citation-coverage-audit` | Find missing classic, closest, benchmark, and recent concurrent citations before submission |
| `citation-audit` | Run a pre-submission audit of LaTeX citation keys, BibTeX entries, metadata, citation claims, labels, and references |
| `submit-paper` | Run a pre-submission readiness check for a LaTeX paper project, including source formatting, local layout debugging, source hygiene, and the configured compile backend without recording one user's local TeX availability |
| `model-card-writer` | Generate model cards, dataset datasheets, reproducibility statements, and artifact READMEs for model releases and venue-required documentation |
| `release-code` | Prepare and publish a research code repository for public release |

## Adding or Updating a Skill

Skills live in `skills/<skill-name>/` and must include a `SKILL.md` with YAML frontmatter:

```markdown
---
name: skill-name
description: One sentence describing when to use this skill. Include trigger phrases.
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Skill Title

Brief description.

## When to Use This Skill
...

## Steps
...
```

Guidelines:
- The directory name must exactly match the `name` field.
- `name` must use lowercase letters, numbers, and hyphens only.
- Keep `SKILL.md` under 500 lines; move large references into linked files.
- Use `scripts/` for shell automation so the agent can run commands without embedding large shell blocks inline.
- Use `templates/` for files the skill writes into user projects.
- Write the `description` as a routing rule, not a title. Agents use it to decide when to activate the skill.
- If the user is outside an initialized ML research project and wants to create or enable one, route through `ml-research-bootstrap`; inside an initialized ML research project, route unknown-skill workflows through `ml-research-router` before guessing a leaf skill.
- When a live session repeatedly misses an existing skill, promote the lesson into routing metadata, core contracts, references, templates, wrappers, and project memory; do not leave it only as chat history.
- Keep examples and venue/tooling references aligned with the actual templates and scripts shipped in the same skill directory.

## Validation

There are no automated tests in this repository. Before manual validation, run the local repository sanity check:

```bash
python3 scripts/validate_skills.py
```

This covers frontmatter, helper references, top-level skill inventory consistency, template placeholder format, and basic Python/shell syntax.

To run the full ml-research-specific test suite, run:

```bash
python3 -m unittest discover tests
```

After the 2026-05-28 hard-slim the suite covers ml-research-specific
skill bundles only: `test_init_python_project_scaffold` and
`test_latex_layout_issue_bundler`. Tests for skills now owned by sibling
packs (code-reviewer / memory-publication-auditor / reference-* /
remote-command-* / run-status-monitor / sidecar-task-runner /
token-usage-auditor / project-push) were removed and should be re-homed
in their pack repos.

**Hub-layer tests** (kernel schema, install/repo-split handoff contract,
install plan validator, install + repo-split writer previews, applier
refusal + happy paths, profile-routing harness, adapter export) live in
[`skill-os`](https://github.com/a-green-hand-jack/skill-os). Run them there
if you change the kernel schema, the contract, or the installer.

Then validate changes by exercising the skill in the target agent runtime:

1. Install the skill with `npx skills add`, such as `npx skills add a-green-hand-jack/ml-research-skills -g -a codex claude-code -s <skill-name> -y`
2. Invoke it by describing a matching task
3. Check that the generated instructions, scripts, and templates behave as intended

## Session Start Protocol (for ML research projects using these skills)

**Step 0 — Detect scope first** (before reading any memory):

```bash
git rev-parse --show-toplevel    # is this a worktree or the project root?
git rev-parse --git-common-dir   # if different from <show-toplevel>/.git → you are in a worktree
```

If **inside a code-worktree** (e.g. `<Project>/code-worktrees/<branch>/`):

1. Read `.agent/worktree-status.md` in the current worktree — local purpose, latest in-progress result
2. Read `<ProjectRoot>/memory/BRIEFING.md` — project-wide context
3. Read `<ProjectRoot>/memory/project-conventions.md` — active rules for this project (wrappers, server paths, scope rules); treat Active rows as binding
4. Read `<ProjectRoot>/memory/hot-results.md` — confirmed project-level results
5. For Python work, use the shared code uv env by default: `UV_PROJECT_ENVIRONMENT=<ProjectRoot>/.uv-envs/code`; run through `uv run` from the active worktree, and create a stage/worktree env only for dependency or stack changes with the reason recorded
6. Write in-progress results to **worktree `.agent/`**, not to `ProjectRoot/memory/`
7. Graduate a result to `ProjectRoot/memory/hot-results.md` only when it is confirmed and directly changes a project claim

If **at the project root** (contains `memory/`):

1. Read `memory/BRIEFING.md` first
2. Read `memory/project-conventions.md` — active rules the agent must follow now
3. Read `memory/hot-results.md` before any experiment-related decision
4. Read `memory/current-status.md` for full detail
5. Re-verify volatile facts (git state, job queues, running processes)

At session end, regenerate `memory/BRIEFING.md` and update `memory/hot-results.md` only for confirmed, cross-component results. This ensures the next session starts with accurate state.

## Notes for Agents

- Prefer updating the minimum set of files needed when a skill changes.
- If you change a skill's behavior, also check whether `README.md` and `CLAUDE.md` now need corresponding updates.
- Treat `SKILL.md`, helper scripts, and templates as a single unit; avoid documenting behavior that the shipped assets do not actually support.

---
> Source: [a-green-hand-jack/ml-research-skills](https://github.com/a-green-hand-jack/ml-research-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
