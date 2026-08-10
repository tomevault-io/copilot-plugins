## otto

> > How agents work in **this** repo (otto-dev, the framework itself) — not the rules Otto installs

# CLAUDE.md — Agent Operating Rules

> How agents work in **this** repo (otto-dev, the framework itself) — not the rules Otto installs
> into a target repo. Auto-loaded every session. Keep it tight; promote durable `learnings/`
> entries up here via `/otto-distill`.

## What this repo is

Otto is a **routing and orchestration layer**, not an implementation. It orchestrates three
read-only sources — **SK** spec-kit · **AS** agent-skills · **PK** pocock-skills — and ships the
*install recipe*, never the implementations. See [`sources/sources.md`](sources/sources.md).

There is no build, no runtime, no dependency tree. The artifacts are Markdown, one YAML registry,
and shell/PowerShell scripts. Layers: sources (0) → [`registry/capabilities.yaml`](registry/capabilities.yaml) (1)
→ [`router/`](router/) (2) → [`workflows/spine.md`](workflows/spine.md) (3) → recipes (4).

## Commands

- **Self-consistency:** `bash install/verify-otto.sh` — audits Otto against itself (invocation
  drift, registered-skill drift, stamped-pin drift, dashboard↔registry drift). Run after any
  registry, dashboard, or skill-unit edit.
- **Audit one run:** `bash install/otto-gate.sh specs/NNN` — checks a run's gate trail is
  auditable (rows exist, PASS/WAIVED name checkable evidence, no stage stuck on FAIL).
- **Register skill units:** `install/register-skills.{ps1,sh}` — **re-run after editing any
  Otto skill unit**, or the host keeps serving the old copy (known drift lesson;
  `verify-otto.sh` check 7 exists because of it).
- **Per-repo setup:** `install/otto-init.{ps1,sh}` — scaffolds a target repo end-to-end.
- Hooks: `otto-session-start.{ps1,sh}`, `otto-learnings-nudge.{ps1,sh}`.

Scripts come in `.ps1` + `.sh` mirrors — **edit both or neither.** `otto-gate.sh` and
`verify-otto.sh` are bash-only today.

## Conventions

- **`registry/capabilities.yaml` is canonical.** DESIGN.md §5 is the human-readable view of it —
  when they disagree, the YAML wins and DESIGN is the bug.
- **Invocation rendering:** the registry writes spec-kit's dotted form (`/speckit.plan`); Claude
  Code renders it hyphenated (`/speckit-plan`). Emit the hyphenated form to the user; keep the
  dotted form in the registry.
- **Windows encoding (load-bearing):** ship **ASCII-only** `.ps1`; write every file **UTF-8
  without BOM**. A BOM breaks YAML-frontmatter parsing of any `SKILL.md`-like file.
- **Assessments** go in `assessments/YYYY-MM-DD-<slug>.md` with the house shape: meta blockquote →
  Verdict → evidence tables → **Coverage** (what was *not* scanned, explicitly) → Recommendation.
- **Deferred work goes in [`BACKLOG.md`](BACKLOG.md) with an act-on trigger** — an observable
  event, not a date. A row without a trigger is a wish.
- **`learnings/global.md` entries are hints-to-verify**, not facts. Re-check any named file, flag,
  or version before acting on one.

## Boundaries

- **Never auto-commit, push, or stage.** Git is human-run — propose the command, never execute it.
  This holds even when a change is obviously correct.
- **Never edit, copy, or vendor the three sources.** They live at their own paths, pinned by ref
  in `sources/sources.md`. Otto routes to them; it never absorbs them.
- **Never advance a stage without logged evidence** (`specs/NNN/decision-log.md`). "Seems right"
  is not a gate pass — [`router/routing-policy.md`](router/routing-policy.md) §D.
- **Never let the actor grade its own mechanical gate.** Evidence MUST be independently
  re-checked — re-run the command, or a second context reads the report.
- **Never build a deferred layer speculatively.** Build from run evidence: the principle that made
  the Workflow backend work first time, and the standing temptation the 2026-07-02 assessment
  flagged. If a trigger hasn't fired, it goes in `BACKLOG.md`.
- **Always state a substitution** rather than failing silently when a source is absent
  (`sources/sources.md` graceful degradation) — no silent skips.

## Patterns

Gate predicates carry a **class** that sets enforcement ([`workflows/spine.md`](workflows/spine.md)):

- **mechanical** — definite answer (tests green, zero CRITICAL). Auto-evaluated, evidence
  independently re-checked.
- **judgment** — needs weighing (axes covered, confidence reached). Judged by a context separate
  from the actor; low confidence → pause, don't guess.
- **human** — belongs to the human (sign-off, go/no-go). Always a pause point, in every mode.

When adding a gate, tag its class first — the class determines who may evaluate it.

---
> Source: [BenRamins/otto](https://github.com/BenRamins/otto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
