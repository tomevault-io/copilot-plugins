## codex-cstack

> Codex workflow stack inspired by gstack, with explicit modes for planning, review, shipping, QA, and retros.


# cstack

Use this meta-skill when the user asks for a structured workflow rather than ad-hoc implementation.

## Sub-skills

- `plan-product-review`: clarify the real user problem, outcome, and scope boundaries before implementation.
- `plan-eng-review`: define architecture, data flow, failure modes, and test matrix before coding.
- `review`: perform bug- and regression-focused code review with severity and concrete references.
- `ship`: prepare a branch for merge with clean checks, clear change notes, and release confidence.
- `qa`: execute a lightweight but explicit validation plan and report evidence plus risk.
- `retro`: summarize wins, misses, and concrete next improvements after shipping.

## Selection guide

- If the user request is about what to build, start with `plan-product-review`.
- If the user request is about how to build safely, use `plan-eng-review`.
- If the user asks for "review", default to `review`.
- If the user asks to finalize and deliver, use `ship`.
- If the user asks to test behavior, use `qa`.
- If the user asks for lessons learned, use `retro`.

---
> Source: [SilentJMA/codex-cstack](https://github.com/SilentJMA/codex-cstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
