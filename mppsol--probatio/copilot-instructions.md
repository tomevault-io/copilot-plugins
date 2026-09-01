## probatio

> Probatio is built by **two agents that cross-review each other**: Claude Code (CC) and Codex.

# AGENTS.md — Probatio two-agent operating contract

Probatio is built by **two agents that cross-review each other**: Claude Code (CC) and Codex.
The repo — not chat memory — is the only shared memory. **Every handoff is a committed artifact**
(task brief, code, review file).

## Division of labour (by task *frame*, not "implement vs review")

The split axis is how much **frame** a task has — goal clarity, convergence condition, the contour
of the right answer.

- **Frame thin** (exploration: vague goal, unknown state, trial-and-error) → **CC**. Manufactures the
  contour while progressing. Owns: architecture, ADRs, task briefs, exploratory spikes, and the final
  "is this explainable / safe to operate?" pass.
- **Frame thick** (convergence: clear diff, fixed perspective, a converging answer) → **Codex**. Fast
  and sharp. Owns: implementing tightly-specified task briefs, tooling, and **adversarial audits/reviews**.

**Product-specific:** Probatio's moat is a verifier that catches what *a different intelligence* does.
So Codex is also the natural **independent red-teamer** against CC's invariants — a genuinely different
model trying to beat the verifier is worth more than CC red-teaming itself (see `STAGE0_DESIGN.md` §0b(2)).

**Cross-pass rule:** whoever implemented a change is NOT its reviewer. The other agent reviews (same
type shares blind spots). A change merges only after a review by the other agent.

## Workflow (brief → branch → review → merge)

1. **Brief.** CC writes a task brief in `docs/tasks/NNN-slug.md` (goal, scope, acceptance criteria,
   out-of-scope, files to touch). A brief is "frame" — make it thick before handing a task to Codex.
2. **Branch.** Always branch from `master`: `task/NNN-slug` (or `claude/...` / `codex/...`). One task
   per branch. Never pile unrelated work onto someone else's branch.
3. **Implement.** The assigned agent implements ON that branch and commits small and often.
4. **Review.** The OTHER agent reviews the branch diff and writes `reviews/NNN-slug.md` (verdict:
   APPROVE / CHANGES, prioritized P0/P1/P2 findings). Be specific and adversarial: missing tests,
   leaky/misnamed abstractions, untested error paths, correctness. Iterate until APPROVE.
5. **Merge.** Only an APPROVED branch merges to `master`. **No agent merges its own un-reviewed work.**

**Review surface — GitHub PRs preferred.** This repo has a private remote (`origin`). Push branches and
review with `gh pr create` / `gh pr diff` / inline comments. (Local fallback: branch + commit + exchange
`reviews/NNN-slug.md` files.)

## The contract (neither agent changes these unilaterally)

The cross-module API is the contract — changing it needs a brief/ADR both agents see:
- `src/policy.rs` — `Policy` trait, `Action`, `AgentAccountRef`, `Observation`, `AgentClaim`.
- `src/verifier.rs` — `StateSnapshot`, `Invariant`, `Verdict`, `Finding`, `FindingKind`, `ShortcutReport`.

## Gates (must hold before review is requested)

- `cargo build` clean, no new warnings.
- `cargo test` green (currently 6 tests). New branching logic ships with tests; live-API code must stay
  testable offline (no test may hit the network).
- No secrets committed. `ANTHROPIC_API_KEY` lives in the environment, never in the repo.

## Git hygiene

- Branch from `master`; rebase onto `master` (not merge commits) to stay current.
- Commit as `psyto <saito.hiroyuki@gmail.com>`.
- The remote is **private** — keep it private; do not change visibility without being asked.

## Running Codex (repo is at /Users/hiroyusai/src/probatio; codex is not on PATH)

```bash
CODEX=/Applications/Codex.app/Contents/Resources/codex
# Review (read-only):
"$CODEX" exec -C /Users/hiroyusai/src/probatio -s read-only "<review prompt: branch + reviews/NNN file>"
# Implement on a branch (writes):
"$CODEX" exec -C /Users/hiroyusai/src/probatio -s workspace-write -o /tmp/codex-out.md "<task brief ref>"
```

## Project context

Probatio stress-tests autonomous on-chain-finance agents on a real rdk episode. Modules:
`world.rs` (episode driver over rdk pure crates), `verifier.rs` (invariant-set-driven shortcut
detector — the moat), `redteam.rs` (discovery loop + parametric attack), `agent.rs` (Claude agent
behind the `Policy` trait), `llm.rs` (Anthropic Messages API via `curl`). See `README.md` and
`STAGE0_DESIGN.md` for the staged roadmap.

---
> Source: [mppsol/probatio](https://github.com/mppsol/probatio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
