## auto-bmad

> This repo is a **BMAD standalone module** (one skill + a Claude `marketplace.json`). The skill

# CLAUDE.md — working in the auto-bmad repo

This repo is a **BMAD standalone module** (one skill + a Claude `marketplace.json`). The skill
(`auto-bmad`) is an orchestrator that runs the full BMAD story workflow one story at a time, on
**Claude Code or Codex**. This file is guidance for working **on the module**, not for using it.

## Core principle (do not violate)
The orchestrator **delegates BMAD work and reports** — it must never implement story work or run
`/bmad-*` skills directly. Every BMAD step (create-story, dev-story, code-review, TEA, retro,
project-context bootstrap/refresh) runs in a delegated `ab-*` sub-agent. **Preserve this separation
when editing.**

The orchestrator owns a small set of actions **directly** (never delegated) — all git/finalize
bookkeeping it already holds full pipeline context for. Don't "fix" these into delegated steps:
- **Git/PR work** — preflight, branching, per-phase commits, push, PR, and the Phase 9 merge prompt.
- **Phase 0 project-context probe** — an existence check that only decides whether Phase 2's
  bootstrap sub-step runs (the bootstrap itself is delegated like every other skill call).
- **Phase 9 finalize writes** — BMAD-status flip to `done` + the pre-push pipeline-report commit.
- **Phase 8 deferred-work archive** — at epic-end, after the project-context refresh, move
  fully-resolved entries out of the active `<impl>/deferred-work.md` ledger into the sibling
  `deferred-work-resolved.md` archive. No `/bmad-*` skill prunes the ledger, and the orchestrator
  already writes this file directly at Phase 7 — so this is connective bookkeeping, not a delegate.
- **Phase 7 HITL-halt feedback** — at the end-of-loop human halt, when the user added
  external-review changes, the orchestrator reads that diff directly to give a brief feedback
  summary before continuing (a lightweight read, **not** a delegated review — the one place it
  inspects code under any tier) and commits it like any other phase.

The mechanics of these live in the reference docs — **don't restate them here**: `git-and-pr.md`
(branching, push, PR, merge prompt) and `pipeline.md` (Phase 0 probe, Phase 7 HITL-halt feedback,
Phase 8 deferred-work archive, Phase 9 status flip + report commit). The only other time the
orchestrator does delegated step work itself is the `inline` delegation tier (see
`delegation-runtime.md`), and even then it follows the same phase contract.

## Delegation is tiered (the heart of the module)
BMAD abstracts neither sub-agent delegation nor per-agent model/effort, so we supply those with
tool-native files and degrade gracefully:
- **Tier 1 `custom-subagents`** (Claude Code, Codex) — each step runs in an isolated delegate at the
  profile's tuned model + effort (Claude `.claude/agents/ab-*.md`; Codex `.codex/agents/ab-*.toml`).
- **Tier 2 `general-subagents`** — generic subagents, no effort knob (effort not honored).
- **Tier 3 `inline`** — no subagents; run the step in-context (documented last resort).

`assets/agents/profiles.yaml` is the **single source of truth** (per-profile, per-tool model+effort
plus tool-neutral persona strings); `phase_profiles` maps each phase to a profile; and
`scripts/render-agents.py` generates the tool-native files from it. Host/mode are `auto` and
re-detected every run, so one provisioned project runs under either tool with no reconfiguration;
`target_tools` only controls which agent files get generated. Full detail: `delegation-runtime.md`
(host detection + the tiers) and `state-and-resume.md` (config/profiles schema, first-run).

## Layout & where behavior lives
- `.claude-plugin/marketplace.json` — Claude distribution (lists the single `./auto-bmad` skill).
- `auto-bmad/SKILL.md` — orchestrator entry point (On-activation gate + procedure). Keep it thin.
- `auto-bmad/references/` — where the real detail lives; each file owns one area:
  - `pipeline.md` — per-phase playbook.
  - `delegation.md` — exact per-skill prompts (tool-agnostic).
  - `delegation-runtime.md` — host detection + the three spawn tiers.
  - `tea-policy.md` — TEA risk rubric / selection.
  - `git-and-pr.md` — branching, commits, push, PR, merge prompt.
  - `state-and-resume.md` — config/state schema, first-run, profiles.
  - `overrides.md` — invocation-override vocabulary.
- `auto-bmad/assets/agents/profiles.yaml` — the single per-profile source (model/effort + persona
  strings). `claude/agent.md.tmpl` + `codex/agent.toml.tmpl` — one shared body template per tool the
  renderer fills in, so the four `ab-*` personas can't drift between tools.
- `auto-bmad/assets/module.yaml` + `module-help.csv` + `module-setup.md` — module identity,
  capability registry, and the self-registration/provisioning flow.
- `auto-bmad/scripts/` — dependency-free helpers, each with a `--self-test` and a self-documenting
  docstring (read the script for exact behavior):
  - `story_plan.py` — sprint-status reader.
  - `state_plan.py` — auto-bmad `state/{key}.yaml` reader (resume detection).
  - `render-agents.py` — agent generator from `profiles.yaml`.
  - `config_plan.py` — detects and additively heals drift between the shipped `profiles.yaml` and a
    project's runtime `config.yaml`.
  - `review_findings.py` — Phase 7 reconciliation reader for a story's `### Review Findings`.
  - `merge-config.py` + `merge-help-csv.py` — config/CSV merge (BMAD template; PyYAML via the
    installer's environment).
- **Repo-root tooling, NOT shipped in the skill:** `CHANGELOG.md` (hand-maintained),
  `scripts/bump-version.py` (release helper — see "Releasing"), `skills/reports/` (tracked
  module-validation snapshots), `docs/` (placeholder).

## Testing
```bash
# Deterministic cores:
python3 auto-bmad/scripts/story_plan.py --self-test
python3 auto-bmad/scripts/state_plan.py --self-test
python3 auto-bmad/scripts/render-agents.py --self-test
python3 auto-bmad/scripts/config_plan.py --self-test
python3 auto-bmad/scripts/review_findings.py --self-test
# Maintainer-only skill (tracked under .claude/ via gitignore exception; NOT shipped to users):
python3 .claude/skills/auto-bmad-compat-check/scripts/bmad_compat.py --self-test
# Marketplace manifest is valid JSON:
python3 -m json.tool .claude-plugin/marketplace.json >/dev/null
# Module structure passes the BMAD validator (run from the repo root, which holds the one skill):
python3 .claude/skills/bmad-module-builder/scripts/validate-module.py .
# Live: add this repo as a local marketplace (Claude) or BMAD module source, install, run
# /auto-bmad in a BMAD project. `/auto-bmad reprovision` re-renders agents after editing profiles.
# Release helper:
python3 scripts/bump-version.py --self-test
```

## Releasing
The version lives in **three** tracked files that must stay in lockstep —
`.claude-plugin/marketplace.json` (`version`), `auto-bmad/assets/module.yaml` (`module_version`),
and the README shields badge. "Publishing" is just **pushing a `vX.Y.Z` git tag** (the BMAD
installer keys upgrade detection off stable tags; the Claude plugin marketplace reads the manifest
`version`).

Cut a release from a clean `main`:
1. Ensure this release's notes are under `## [Unreleased]` in `CHANGELOG.md`, grouped under
   Keep-a-Changelog headings. Write them by hand as changes land — never auto-generate from commits.
2. `python3 scripts/bump-version.py <patch|minor|major>` (or an explicit `X.Y.Z`; `--dry-run` to
   preview). It refuses an empty `[Unreleased]`, guards against version drift across the three files,
   promotes the changelog (date + compare links), rewrites all three versions, then commits
   `chore(release): vX.Y.Z` and tags it.
3. `git push --follow-tags`.

`.github/workflows/release.yml` then fires on the `v*` tag and creates the GitHub Release from that
tag's CHANGELOG section (idempotent; it verifies the tag agrees with all three version files and the
changelog first). That's the only CI — no build/publish step, and nothing re-renders agents on bump
(`/auto-bmad reprovision` is a runtime concern, not a release artifact).

## Conventions
- Conventional Commits (`feat:`/`fix:`/`docs:`/`test:`/`chore:`/`refactor:`).
- Never commit the local BMAD test install or generated agents — `_bmad/`, `_bmad-output/`,
  `.agents/`, `.claude/`, `.codex/` are gitignored. The published repo is module + marketplace +
  docs only. **One deliberate exception** (a `.gitignore` negation): the tracked maintainer skill
  `.claude/skills/auto-bmad-compat-check/` — checks new BMAD releases (npm `latest`/`next`) for
  impact on auto-bmad and offers to bump the README compat markers. It's repo tooling, not shipped
  inside the module (like `scripts/bump-version.py`); everything else under `.claude/` stays ignored.
- Markdown reference files are read by the orchestrator at runtime; keep them concise and
  unambiguous (they are instructions, not prose). Helper scripts stay dependency-free with a
  `--self-test`.
- Don't land a user-facing change without a `CHANGELOG.md` note under `## [Unreleased]` (right
  Keep-a-Changelog heading) in the same commit/PR. Never bump the version files by hand — use
  `scripts/bump-version.py` so all three stay in sync (see "Releasing").
- **Changelog entries are written to be skimmed.** A reader must grasp a release from the **bold
  lead lines alone**, in seconds. Enforce:
  - **One change = one bullet** under one heading. Never bundle (if you're writing "three
    reinforcing fixes", that's three bullets).
  - **Bold headline first, ≤ ~12 words, stating the user-visible effect** ("X no longer Y"), not the
    internal mechanism.
  - **At most ~2 sentences of detail** after the headline — the one fact a reader needs. No
    "Previously…/the gap was…/chicken-and-egg" debugging narrative; the *how* lives in the reference
    docs and the commit body.
  - **No inline file-touch lists** — git history records touched files. If a pointer genuinely
    helps, one terse trailing parenthetical (`(pipeline.md, git-and-pr.md)`), never woven into
    sentences.
- **Past released `## [X.Y.Z]` sections are immutable** — apply this style to new entries only;
  never rewrite a shipped section (`release.yml` renders the GitHub Release from it).

## Known platform facts (verified)
- **Claude Code:** sub-agents take `model:` + `effort:` frontmatter (effort is settable ONLY
  there, not via the Agent tool — hence the templates); they CAN invoke skills but CANNOT spawn
  sub-agents. `.claude/agents/` is scanned into the invokable roster **only at process launch** —
  agents rendered mid-session (first-run setup, reprovision) aren't invokable until a full quit &
  relaunch (`/clear` reuses the same process and does not re-scan).
- **Codex:** subagents are TOML files in `.codex/agents/` (project) or `~/.codex/agents/`, with
  `model` + `model_reasoning_effort` (gpt-5.x effort: low|medium|high|xhigh — xhigh is the ceiling);
  invoked by naming the agent in natural language — Codex spawns/collects them. Model names are
  environment-specific (retunable per install), so they're config, not hardcoded — the shipped
  defaults are real.
- **BMAD** has no portable abstraction for delegation or model/effort; modules are skills copied
  into a tool's skills dir (`.claude/skills/`, `.codex/skills/`). Hence the tiered design.
- **BMAD update of a custom-source module (`abm`):** `--action quick-update` only re-pulls modules
  cached under `~/.bmad/cache/` and **skips custom-source re-cloning entirely** (`installer.js
  quickUpdate` adds a custom module only if `findModuleSourceByCode` hits a cached repo). And
  `resolveInstalledModuleYaml` never searches the project tree, so a self-registered/
  marketplace-installed `abm` shows `source: unknown` in `_bmad/_config/manifest.yaml` and emits
  benign `could not locate module.yaml for 'abm'` warnings on every update. Fix: re-supply the
  source — `npx bmad-method install --action update --custom-source <repo-url> --yes` (re-clones,
  rewrites the manifest source). So the README "Updating" section must recommend `--action update
  --custom-source …`, **never** bare `quick-update`.
- `/bmad-create-story` has no `validate` mode; it self-validates against its checklist.
- **Shell globs:** the orchestrator's probe commands run under whatever shell the host uses (zsh,
  fish, bash). An unmatched glob is fatal in zsh/fish (`nomatch` ⇒ exit 1), and the `for f in
  *.glob; do …` loop syntax isn't even portable to fish — so probes must not iterate raw globs.
  Use `find … -name '<pat>'` (external binary, empty output + exit 0 everywhere) or Python.

---
> Source: [stefanoginella/auto-bmad](https://github.com/stefanoginella/auto-bmad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
