## leray-hopf

> Operating rules for the **AI agent team** working in this repository (roster:

# AGENTS.md

Operating rules for the **AI agent team** working in this repository (roster:
`docs/agent-roles.md`) — this is not a policy imposed on human contributors; see
`CONTRIBUTING.md` for that audience. This is a Lean 4 + mathlib formalization of
Leray–Hopf weak existence. Mathematical scope is fixed by the plan files in
`docs/`; this file governs *how agents work*, not *what* to prove.

This is the **canonical, audience-neutral source of truth for the agent team**,
shared identically by every agent: Claude Code loads it through `CLAUDE.md`
(`@AGENTS.md`), and external reviewers (Codex) read it directly. "Audience-neutral"
means the same rules bind every agent tool equally — it does not mean this
document is addressed to human contributors. A human contributor should read
`docs/guardrails.md` for the substantive discipline rules that bind any change
(no overclaiming, no silent axioms, …); this file's tooling mandates (e.g. hard
rule 1's `agent-preflight.sh` requirement below) are agent-team operating
procedure specifically, not something asked of a human contributor. Keep it
compact; put depth in the linked `docs/` files. Claude-only orchestration notes
live in `CLAUDE.md`, not here.

## Before you edit

Run the preflight and read the source-of-truth chain:

```bash
bash scripts/agent-preflight.sh
```

Source-of-truth order (higher wins): current task instruction → this file → existing Lean
theorem statements → README/docs. (The original scope-setting plan files,
`docs/archive/milestone.md` and `docs/archive/leray_hopf_lean_mvp_plan.md`, are archived and
historical — both capstones are long past their MVP scaffold, which was deleted in issue #144;
see `docs/architecture.md` for the current module layout.)

## Hard rules

1. **Run preflight before and after editing.** Do not report success without a green `lake build`.
2. **Do not rename a theorem target** unless explicitly instructed.
3. **Do not weaken a statement to make a proof pass** — no dropping hypotheses, no
   narrowing conclusions, no strengthening assumptions to trivialize.
4. **Do not replace a theorem by `True`, `Nonempty`, or any vacuous proposition**
   unless the file is explicitly marked scaffold-only.
5. **No new `axiom`, `constant`, `opaque`, or `unsafe`** without a same-line
   `-- ALLOW_AXIOM: <reason>` marker and an entry in the file's assumptions section.
6. **No analytical assumption hidden in a theorem name.** The name must describe what
   is actually proved. Reserved overclaim terms are blocked by CI.
7. **No unmarked `sorry`.** An incomplete proof needs `-- ALLOW_SORRY: <reason>` on its line.
8. **If a proof is not possible, leave the statement intact** and add a precise
   `-- TODO:` describing the exact blocker. Do not patch around it by weakening math.
9. **Prefer small local lemmas** over large monolithic tactic blocks.
10. **Do not import broad modules unnecessarily;** justify a heavy import.
11. **Do not stream full Lean build logs into agent context;** capture logs to a file
    and inspect only the exit status plus relevant tail/error lines.

## Edit ownership (see `docs/agent-roles.md`)

- Proof bodies (`:= by …`) — `lean-prover` only.
- Signatures, structure, imports, lakefile — `lean-coder`.
- `docs/` and planning artifacts — `lean-planner`, `sot-researcher`.
- Reviewers and researchers do not edit Lean sources.

## Every PR / handoff must report

- files changed,
- theorem/def names added,
- remaining `sorry` (count + locations),
- new assumptions (`axiom`/`opaque` added, with justification),
- whether `lake build` passed.

## Details

- Discipline rules and rationale: `docs/guardrails.md`
- Operational protocol (failure handling, reporting): `docs/agent-protocol.md`
- Running checks: `docs/build-and-checks.md`
- Team roles, contracts, and the Codex review protocol: `docs/agent-roles.md`

---
> Source: [uda-lab/leray-hopf](https://github.com/uda-lab/leray-hopf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
