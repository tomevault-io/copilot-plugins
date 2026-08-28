## avo

> This file is read automatically by Codex CLI and other agents that follow the

# AGENTS.md — instructions for coding agents working in this repo

This file is read automatically by Codex CLI and other agents that follow the
`AGENTS.md` convention. Claude Code users additionally have a `/avo` skill in
`.claude/skills/avo/`.

There are two distinct things an agent does here, and they need different rules.

---

## A. Working *on* the AVO codebase

Ordinary software work: fixing bugs, adding targets, adding backends.

```bash
python3 -m venv --system-site-packages .venv    # if Python is externally managed
.venv/bin/pip install -e ".[dev]"
.venv/bin/pytest -q                             # 48 tests, no model or network needed
.venv/bin/avo doctor
```

Conventions:

- **No mandatory dependencies beyond PyYAML.** Anything else goes in an extra
  and is imported lazily, so `avo doctor` can report what is missing rather
  than the import failing.
- **The framework never calls a model.** Session mode is the default precisely
  because it costs nothing. Keep it that way.
- **Prompts are the framework/agent interface.** `src/avo/prompts.py` changes
  behaviour more than most code does. If you edit it, say in the commit message
  what you expect the operator to do differently.
- **Test anything touching the commit policy, the correctness gate, or the
  lineage.** Those three are what make results trustworthy.
- `docs/PAPER_MAP.md` maps every section of the paper to the code. If you change
  something it describes, update it.

---

## B. Acting *as* the variation operator

This is the interesting one. You are the `Agent` in `Vary(P_t) = Agent(P_t, K, f)`:
AVO owns the lineage, the scoring, and the commit policy; you own the thinking.

Use the `avo_*` MCP tools if they are available, otherwise the CLI — the
operations are identical. Full protocol in [`docs/DRIVING.md`](docs/DRIVING.md).

### The loop

1. `avo_start_run` / `avo start --target <name> --max-steps N`
2. `avo_next_step` / `avo prompt` — read it properly, it changes every step
3. Edit files under `work/`; read `kb/` when you need it
4. `avo_evaluate` / `./avo-eval` — as often as you like, it does not touch the lineage
5. `avo_submit -m "what changed and its measured effect"`
6. On a reported stall: `avo_supervisor_brief`, answer it yourself, then
   `avo_record_supervisor`
7. Repeat, then `avo_plot`

### Rules that keep a run honest

- **One substantial change per step.** Bundling makes the result unattributable
  and breaks the ablation.
- **Measure before submitting.** A change you did not measure is a change you
  did not make. If you know it will be rejected, fix it or `avo_revert` — ending
  a step on a measured regression wastes it.
- **Never edit the lineage.** No `git commit`, `git reset`, or `git checkout`
  inside `work/`. Reading history (`git log`, `git show v7:file`,
  `git diff v6 v7`) is expected and encouraged.
- **Never touch the evaluator or the target definition to make a score go up.**
  `targets/*/eval.py`, `target.yaml`, and the canonical engine or reference
  implementation are out of bounds during a run. Editing the scoring function is
  not an optimisation, it is a broken experiment.
- **Do not special-case the benchmark inputs.** The seeds and shapes are fixed so
  that versions are comparable, not so they can be memorised. Prefer changes
  that are defensible as general behaviour.
- **Profile before optimising.** Guessing the hot path has a poor track record.
- **Maintain `NOTES.md`,** especially dead ends. There is no next step for you,
  only a new session reading what you left.
- **Report honestly.** If the score went down, say so with the numbers. A run
  where three steps in ten are accepted is normal.

### Reading the score

`f` returns a vector — one number per benchmark configuration — plus a hard
`correct` gate. A candidate that fails the gate scores zero however fast it is.
Per-configuration movement is diagnostic; the geometric mean alone hides it.

Treat small score moves as noise unless a mechanism explains them. In the worked
example (`examples/game2048-run/`), a change with provably identical search
semantics moved the aggregate by 4% purely through floating-point tie-breaking.

---

## Repository layout

```
src/avo/           the framework — nothing here knows about kernels or games
  prompts.py       the whole framework/operator interface
  run.py           run state, commit policy, trajectory
  session.py       driver: the session you already have is the operator
  loop.py          driver: unattended, spawns an agent per step
  mcp_server.py    the same operations as MCP tools (no dependencies)
  agents/          backends for unattended mode
targets/           domains: seed program, knowledge base, evaluator
examples/          a complete worked run, including its dead ends
docs/              PAPER_MAP.md, TARGETS.md, DRIVING.md
```

---
> Source: [gatordevin/avo](https://github.com/gatordevin/avo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
