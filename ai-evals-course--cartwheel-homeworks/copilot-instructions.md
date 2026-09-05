## cartwheel-homeworks

> This repository holds the cumulative student work for the course "Evaluating and Improving AI Agents." It contains one customer facing support agent over the seeded fictional Cartwheel platform. Students continue in the same repository across all five modules.

# Cartwheel course repository

This repository holds the cumulative student work for the course "Evaluating and Improving AI Agents." It contains one customer facing support agent over the seeded fictional Cartwheel platform. Students continue in the same repository across all five modules.

## Commands

- Install: `uv sync`
- Seed the world (deterministic, dev scale): `uv run python -m seed.generate`
- Tests: `uv run pytest` (offline, no API keys needed; homework tests are xfail until implemented)
- Chat: `uv run python -m agent.cli --role shopper`
- Serve: `uv run uvicorn server.app:app --port 8010`
- Tracing stack: `docker compose -f observability/docker-compose.yml up -d`
- CI evaluation suite (Module 3; complete agent tasks need an API key): `harbor run -p eval_cases/`
- Replay one failing task ~10 times: `harbor run -p eval_cases/e-001-unconfirmed-write/ --repeat 10`
- Leakage check (task inputs vs. prompts): `uv run pytest tests/test_leakage.py`

## Layout

- `SPEC.md` is the support agent specification with the access matrix and the criteria table.
- `facts.yaml` is the facts sheet; every policy number comes from it.
- `seed/` generates `data/cartwheel.db` and `data/policies/`. Do not edit generated data by hand.
- `agent/` contains the support agent. `agent/tools.py` holds the Homework 1 holes.
- `agent/guards.py` holds the Module 4 guard holes: the detection functions
  (`flags_injection`, `strip_external_links`), the wired SDK guardrails
  (`injection_input_guardrail`, `link_output_guardrail`), and the
  `needs_approval` predicate (`refund_needs_human`). `agent/approvals.py` holds
  the human-approval flow that fills the Module 1 stub: four holes
  (`list_pending_refunds`, `approve_refund`, `reject_refund`,
  `render_decision_record`); `ensure_approvals_table` is provided.
  `agent/review.py` is the review-surface CLI with a loop seam the student
  fills. `agent/killswitch.py` is the provided Module 4 kill switch, wired into
  the refund and cancel tools.
- `seed/adversarial.py` injects the Module 4 adversarial fixtures post-seed
  (store 21, user 5012, order #7002, a poisoned help-center page, a poisoned
  refund reason). It is never run by `make seed`; load it by hand for live
  red-teaming.
- `tests/test_adversarial.py` is the Module 4 `-k m4` suite: invariant
  code-block tests that pass out of the box, plus the guard, approval, and
  predicate holes. `promptfooconfig.yaml` is the Artifact F red-team starter, a
  basic config the student grows into a comprehensive one in homework Part C.
- `homework/` contains the student assignments for Modules 1 to 5. Instructor notes remain outside the Cartwheel repository.
- `observability/instrument.py` contains the Homework 2 tool result span holes.
- `scenarios/` contains the synthetic data skill and the scenario runner.
- `analysis/` contains the Module 2 error analysis skill.
- `eval_cases/` is the Module 3 evaluation case set (`cases.jsonl`; schema in `eval_cases/README.md`).
- `tests/eval/` holds the Module 3 evaluation tests; `tests/eval/passk.py` holds `hw6` holes.
- `replay/` contains the replay code, and `replay/harness.py` holds `hw6` holes.
- `monitoring/` contains the CD monitoring job, and its three helper files hold `hw7` holes.
- `scripts/check_leakage.py` compares evaluation inputs with prompts and contains an `hw6` hole.
- `.github/workflows/evals.yml` runs the offline checks on each push and the complete agent cases plus leakage check on pull requests.

## Rules for coding agents working here

- Homework holes are marked `### YOUR CODE HERE (HWn)` (or `(m2)` /
  `(m4)`) and raise `NotImplementedError`. Implement exactly what each
  docstring specifies. The Module 4 holes (`m4`) live in `agent/guards.py`,
  `agent/approvals.py`, and `agent/review.py`, and are selected by
  `tests/test_adversarial.py -k m4`.
- The Module 4 kill switch reads the `CARTWHEEL_KILL_SWITCH` env var: "off"
  (default, a no-op), "refunds" (pauses the refund tool), "readonly" (pauses
  all write tools). It is provided; do not weaken it. Keep the default a
  no-op so existing tests stay green.
- Do not edit `seed/adversarial.py` outputs into `data/`. The adversarial
  fixtures are injected post-seed into a throwaway copy or a dev DB, never
  committed and never run by `make seed`.
- Do not delete evaluation cases when a bug is fixed; an evaluation case stays as the
  regression test that proves the fix holds. Do not quote evaluation inputs in
  any prompt (the leakage check fails the build).
- Do not weaken or bypass permission checks in `agent/auth.py` or the refund threshold logic.
  Authorization is not a prompt, and it is not negotiable in code either.
- Do not edit `seed/` outputs or the pinned demo orders (#4127, #3980, #4455).
- Never print or commit values from `.env`. Refer to keys by env var name.
- Keep functions small and typed. Return structured tool results
  (`{"ok": ..., "error": ..., "reason": ...}`), never prose errors.

---
> Source: [ai-evals-course/cartwheel-homeworks](https://github.com/ai-evals-course/cartwheel-homeworks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
