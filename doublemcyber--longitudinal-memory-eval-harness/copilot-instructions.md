## longitudinal-memory-eval-harness

> **Done** — `pytest -q` exits 0 against the PRD §11 acceptance suite (A1–A7: frozen adapter contract + 3 baselines run end-to-end, 5 category generators with exact gold labels, all 7 metrics emitted in the scorecard, A4 discrimination holds, A5 determinism holds).

# CLAUDE.md — Longitudinal Memory Eval Harness

**Done** — `pytest -q` exits 0 against the PRD §11 acceptance suite (A1–A7: frozen adapter contract + 3 baselines run end-to-end, 5 category generators with exact gold labels, all 7 metrics emitted in the scorecard, A4 discrimination holds, A5 determinism holds).

**Workflow** — Stages = PRD §13: (1) frozen adapter contract + scaffold + the acceptance suite itself, (2) baselines, (3) generators, (4) metrics + scorecard, (5) runner + discrimination/determinism, (6) real-logs path + stubs. No tests exist yet, so Stage 1 *writes* the gate; thereafter run the gate after every stage and commit only when it passes.

**Verify** — Before marking any stage done, run a separate reviewer pass on Opus 4.8 against the PRD §11 criteria for that stage. Never mark a stage done on your own say-so.

**Git** — Work on `claude/mem-eval-harness`; never commit to or push `main`; one commit per passing stage, conventional message (e.g. `feat: stage 3 category generators`).

**Guardrails** — Stay inside this repo; no `git push`, no force-push, no deleting files outside the working tree. PRD.md is the source of truth — never edit it to make the gate pass.

**Stop** — If the gate can't pass after a fair attempt, stop and write the blocker (stage, failing test, what you tried) to `PROGRESS.md` rather than looping.

---

Launch the loop with:

```text
/goal Loop until `pytest -q` exits 0 with the full PRD §11 acceptance suite present (5 categories, 3 baselines, 7 metrics, A4 discrimination + A5 determinism) AND `git status` is clean on branch claude/mem-eval-harness with one commit per passing stage. Turn cap: 40. If the gate can't pass after a fair attempt, stop and write the blocker to PROGRESS.md.
```

---
> Source: [doubleMcyber/longitudinal-memory-eval-harness](https://github.com/doubleMcyber/longitudinal-memory-eval-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
