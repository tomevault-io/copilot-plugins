## techneering

> > Open-source spec-driven skills plugin. Works on both **Codex** and **Claude Code**.

# Techneering

> Open-source spec-driven skills plugin. Works on both **Codex** and **Claude Code**.

## Plugin Identity

| Field | Value | Notes |
|------|---|------|
| Repository | `github.com/devnomad-byte/techneering` | Open-source repo |
| Plugin name | `tn` | Both Codex and Claude Code identifier |
| Skill call prefix | `tn:` | Prefix for all skills (e.g. `tn:compass`) |

## How Techneering Works With Codex

Codex reads this `AGENTS.md` before doing any work, and loads Techneering's skills from `./skills/`. A `SessionStart` hook (in `hooks/hooks.json`, which Codex auto-discovers by default) also injects the `tn:compass` gateway at session start, mirroring the Claude Code experience. If the hook doesn't fire for any reason, the rules below still apply.

**Why there's both `AGENTS.md` and `CLAUDE.md` (no conflict):** Each host loads only its own file. Codex auto-loads `AGENTS.md` (this file); it does **not** auto-load `CLAUDE.md`. Claude Code auto-loads `CLAUDE.md`; it does **not** auto-load `AGENTS.md`. They serve the same purpose for two different hosts, so both ship in the repo. The `hooks/hooks.json` is shared — Codex injects `CLAUDE_PLUGIN_ROOT` for compatibility, so the one hook file works on both.

**The single most important rule:** Before responding to any user request, check whether a Techneering skill applies — and if it does, follow that skill. Start every session by consulting `tn:compass` to pick the right development path.

## Skill Inventory

| Skill | Call | Type | Description |
|------|--------|------|------|
| Compass | `tn:compass` | Gateway | Auto-injected on session start, selects development path |
| Scout | `tn:scout` | Flow | Standalone exploration mode, pure thinking, writes no files |
| Draft | `tn:draft` | Flow | Generate proposal + specs + design + tasks |
| Forge | `tn:forge` | Flow | Entry point: chains blueprint → assemble |
| Audit | `tn:audit` | Flow | Dual-layer verification: spec compliance + requirement tracing |
| Vault | `tn:vault` | Flow | delta merge → move into archive/ |
| Spark | `tn:spark` | Process | Socratic questioning + project style detection + framework recommendation |
| Blueprint | `tn:blueprint` | Process | Break into bite-sized tasks |
| Diagnose | `tn:diagnose` | Rigid | 4 phases: root cause → pattern → hypothesis → fix |
| Gate | `tn:gate` | Rigid | Iron law: run command → read output → claim → show menu |
| Redgreen | `tn:redgreen` | Rigid | Red-green-refactor cycle |
| Assemble | `tn:assemble` | Execution | Subagent dispatch + two-stage review + parallel scheduling |
| Inspect | `tn:inspect` | Execution | Global cross-task review: consistency, architecture, security |
| Isolate | `tn:isolate` | Execution | Create isolated workspace + verify test baseline |
| Ship | `tn:ship` | Wrap-up | Branch integration: merge / PR / keep / discard |
| Craft | `tn:craft` | Domain | Auto-stacked when a frontend task is detected |

## Development Paths

| Path | Use case | Flow |
|------|---------|------|
| Complete | Large changes | spark → draft → forge → gate → user chooses next flow |
| Lightweight | Medium changes | draft → forge → gate → user chooses next flow |
| Fix | Urgent bugs | diagnose → redgreen → gate → user chooses next flow |
| Direct | Trivial changes | direct execution → gate |

**After gate passes, present a menu for the user to choose:**
1. Full verification (audit → inspect → ship → vault) — production code
2. Standard flow (audit → ship → vault) — internal projects
3. Quick commit (ship → vault) — small changes/prototypes
4. Verify only (audit) — just check, don't commit

## Skill Transitions

After each skill completes, its Next Step points to the next skill. The full transition rules live in the `tn:compass` gateway skill (`./skills/compass/SKILL.md`).

## Standard Skill Structure

All skills follow a unified structure: frontmatter → core principle → steps → red flags → quick reference → Next Step → guardrails.

Rigid skills (redgreen/diagnose/gate) must have a red-flags list. All skills must have Next Step + guardrails.

## config.yaml Core Fields

On the first draft, `techneering/config.yaml` is generated with two core sections:

| Field | Purpose | Source |
|------|------|------|
| `coding_conventions` | Code style constraints (naming, structure, patterns, formatting) | spark detection / simplified detection / framework defaults |
| `quality_standards` | External quality standards (Alibaba Taishan, PEP 8, etc.) | Recommended after spark detects the tech stack |

**Standard source options:** Java → ali-taishan-java, Python → ali-taishan-python + pep8, Go → gofmt, frontend → user's choice.

**Full-flow injection:** draft writes → assemble implementer/reviewer read and enforce → inspect cross-checks globally.

---
> Source: [devnomad-byte/techneering](https://github.com/devnomad-byte/techneering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
